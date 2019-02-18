# Service Discovery

Kubernetes manages service discovery through a `Service` object. Using a service, Kubernetes will assign a Cluster IP to this service and load balance requests between the pods assigned to the service.

This also addresses an issue with programs (such as Java) that would cache the DNS entry and would not get updated afterward.

More on services: https://kubernetes.io/docs/concepts/services-networking/service/

# Service DNS
By exposing a service, Kubernetes assign a DNS entry so that it can be accessed by other services.

For the previous example, the DNS entry would be `kuard-prod.default.svc.cluster.local`

- `kuard-prod`: the name of the deployment
- `default`: the namespace
- `svc`: the service object
- `cluster.local`: the cluster domain name

Within the same namespace, only the deployment name is required.

# Assigning services
A service can be added to your manifest as a new resource. Resources can be stored within the same manifest or in different ones.

The three dashes `---` are delimiter within a yaml file to split resources.

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kuard
  name: kuard-service
  namespace: default
spec:
  ports:
  - nodePort: 31076
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: kuard
  type: LoadBalancer
```

*kuard-deployment.yml*
``` yaml
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
        env: prod
        ver: "1"
    spec:
      containers:
        - image: gcr.io/kuar-demo/kuard-amd64:1
          name: kuard
          ports:
            - containerPort: 8080
              name: my-port
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /healthy
              port: 8080
            initialDelaySeconds: 5
            timeoutSeconds: 1
            periodSeconds: 10
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            periodSeconds: 2
            initialDelaySeconds: 0
            failureThreshold: 3
            successThreshold: 1
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

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kuard
  name: kuard-service
  namespace: default
spec:
  ports:
  - nodePort: 31076
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: kuard
  type: LoadBalancer
```

To apply your new changes. Note that this update to the manifest will stop the pods and restart them to apply the new configuration.
```bash
kubectl apply -f kuard-deployment.yml
```

This would add a readiness probe that does the following:
- checks the pod every 2 seconds as soon as the pod comes up
- marks the pod as failed after three failed attempts
- marks the pod as ready after one successful attempt

Ready pods are those that are sent traffic to.


You can list your services andd see the IP and ports that are exposed.

```bash
kubectl get services -o wide
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE       SELECTOR
kuard-service   LoadBalancer   10.106.198.147   <pending>     8080:31076/TCP   7s        app=kuard
kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP          1d        <none>
```

Now you can access the service through an URL.

`minikube service kuard-service --url`

**Next step**: [Persistent Volumes](08-persistent_volumes.md)
