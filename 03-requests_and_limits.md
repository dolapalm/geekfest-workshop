# Requests

The `Request` section of a container represent the minimal amount of resource 
that Kubernetes should plan for your container under high load.

It is attached to the container section, not the pod, as it's the container that
gets assigned those resources.

It is important you provide the minimal amount of resource your containers will be
using under high-load as Kubernetes will be using that information for distributing
your pods onto different worker nodes.  Failure to do so may include seeing your pods 
on a 100% cpu usage and/or to see your pods getting re-scheduled (killed) at the busiest moment.

`Request` are part of the `resources` of a container in a manifest (see example below);

More on `Request`: [Resource requests and limits of Pod and Container](
https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container)


# Limits

Limits are also part of the resource planning of a container.  It's the hard-limit 
to set on the container to prevent over-usage.  If a container is trying to reserve more
memory when it's already at the limit value, it would result in a `malloc` operation failure.
For a cpu, the container will only be able to use the amount configured (i.e. 400 millicpu as 
per our example below).


More on `Limits`:  [Resource requests and limits of Pod and Container](
https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container)

```yaml
# Code to add to your container manifest
resources:
  requests:
    cpu: "200m"
    memory: "128Mi"
  limits:
    cpu: "400m"
    memory: "256Mi"
```
To apply the new changes 
```
kubectl apply -f kuard-deployment.yml
``` 

*kuard-deployment.yml*
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuard-deployment
  labels:
    app: kuard
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kuard
  template:
    metadata:
      labels:
        app: kuard
    spec:
      containers:
        - image: gcr.io/kuar-demo/kuard-amd64:1
          name: kuard
          ports:
            - containerPort: 8080
              name: my-port
              protocol: TCP
          resources:
            requests:
              cpu: "200m"
              memory: "128Mi"
            limits:
              cpu: "400m"
              memory: "256Mi"
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
```

**Next step**: [Labels and Annotations](04-labels_and_annotations.md)
