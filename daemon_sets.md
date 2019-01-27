# Daemon Sets
Daemon sets offer a unique style of replication. Unlike replica sets where kubernetes will deploy the requested number of replicas without any focus on which node it is deploying to, daemon sets ensure that each node contains the pod you are defining. Therefore if you add or remove nodes to your cluster, kubernetes will always ensure that that pod is deployed there when that node is up.

*Head to the kubernetes documentation for more on [daemon sets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)*
