# K8s_Kubeproxy
K8s 的代理伺服器調節


# 架構圖


          app.ProxyServer -------> SourceAPI
                                      / \
                                     /   \
                                    /     \
                                   /       \
                                  /         \
                                    Refector
                                      Node
                             End Point     Service
                           /         |     |        \
                          /          |     |         \
                         /           |     |          \
                        /            |     |           \
                       /  Client.EndPoint Client.Service\
                      /              |     |             \
                     /               |     |              \
                  Channel             Master             Channel
                    |               API Server              |
                    V                                       V
                  (merge)       /                 \      (merge)    
                    |          /                   \        |
                    V         /                     \       V
                (broadcast)  /                       \  (broadcast)
                    |                                       |
                    |                                       |
                (OnUpdate) ------>    proxy.LB   <------ (OnUpdate)
