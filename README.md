# K8s_Kubeproxy
K8s 的代理伺服器調節


# 架構圖


此架構圖核心關鍵是 API Server，在其上同步連結端點與服務資訊，為每個服務建立本地端代理，完成負載能力的服務轉發功能。

Source API 由代理伺服器建立，拉取了 API Server 的 EndPonit 和 Service 資訊，分別由 Refector EndPoint 與 Refector Service 實現，透過 Client.EndPoint、Client.Service 的拉取資料產生對應的更新資料放入 Channle 中，最終通道中的資訊遞送到 Proxier，Prixier 為每個服務建立 Socket 實現了服務代理，並且建立相關 NAT 規則於 iptable 上，然後在 LB 上開通該 Service 的 LB 功能。


          app.ProxyServer -------> SourceAPI (2)
                                      / \
                                     /   \
                                    /     \
                                   /       \
                                  /         \
                                  
                                    Refector
                                      Node
                                       (3)
                                       
                             End Point     Service
                             
                           /         |     |        \
                          /          |     |         \
                         /           |     |          \
                        /            |     |           \
                                     |     |
                                       (4)
                          Client.EndPoint Client.Service 
                       
                      /              |     |              \
                     /               |     |               \
                     
                                                           (5)
                  Channel             Master (1)         Channel
                    |               API Server              |
                    V                                       V
                  (merge)       /                 \      (merge)    
                    |          /                   \        |
                    V         /                     \       V
                (broadcast)  /                       \  (broadcast)
                    |                                       |
                    |                                       |  (OnUpdate)
                (OnUpdate) ----> LB.RR -- proxy.LB (8) <-- Proxier (6)
                                          |                 |
                                          V                 V
                                         State         service info -> Socket (7)
                                          |
                 (此結構體稱為狀態機，用於保存此服務的 endpint 位址及目前對話狀態的 affinityPolicy)
