# Namespaces
Namespaces are used to organized objects in a Kubernetes cluster. By default, 
the `kubectl` command will interact with the `default` namespace. You can change 
that namespace by using the `--namespace` or `-n` flag to the `kubectl` command.

You can create new namespaces with the `kubectl` command and interact with it.
```bash
kubectl create namespace my-new-namespace
kubectl get pods -n my-new-namespace
```

More on namespaces: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

# Contexts
Now that you have multiple namespaces, you may get bored of always having to type the namespace in your `kubectl` command.

With kubectl, you can register contexts to save new configurations such as:
- namespace
- credentials
- cluster

To create a new context:
```bash
kubectl config set-context my-context --namespace=my-new-namespace
```

To use your new context:
```bash
kubectl config use-context my-context

kubectl get pods
# Returns the list of pods in my-new-namespace
```

All the configuration is saved in `~/.kube/config`

More on contexts: https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
