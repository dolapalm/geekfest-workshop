# Pods 

> A Pod represents a collection of application containers and 
volumes running in the same execution environment. Pods, not 
containers, are the smallest deployable artifact in a Kubernetes 
cluster. This means all of the containers in a Pod always land 
on the same machine.
>  - *"Kubernetes - Up and Running" page 38*

Pods are the closest representation of a VM.  You can create a resource of type pod 
in any namespace you have.  To do so, you may have to inform Kubernetes as what physical 
resources are required for your application to run; Cpu, Ram, Volumes, labels, etc.

There are multiple ways in which we can deploy pods (*deployment,daemonset,etc.*). 
However, the simplest form is it's imperative form;
```bash
kubectl run kuard-pod --image=gcr.io/kuar-demo/kuard-amd64:1 # this create a deployment resource
kubectl get pods
```

To delete the newly created pods 
```bash
kubectl delete deployment/kuard-pod
```

If you have to be more specific however, the manifest is probably more helpful.

# Manifest

A manifest is a simple YAML text file that is sent to kubernetes.  The YAML file represent the 
desired state that we would like kubernetes to do for us.


*kuard-deployment.yml*
``` yaml 
apiVersion: v1
kind: Pod
metadata:
  name: kuard-pod
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      ports:
        - containerPort: 8080
          name: my-port
          protocol: TCP
```

To run the pod above, run the following command;
```bash
kubectl apply -f kuard-deployment.yml
```

To get more information about the manifest of the pod;
```bash
kubectl describe pods kuard-pod
```

there is 2 way of deleting the pods;
```bash
kubectl delete -f kuard-deployment.yml
```
```bash
kubectl delete pods/kuard-pod
```

More on `Pods`: [Understanding Pods](
https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#understanding-pods)

**Next step**: [Create a deployment](02-deployments.md)
