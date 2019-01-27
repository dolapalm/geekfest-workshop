# Liveness Probe
Kubernetes needs a way to determine whether a container is running properly.
This can vary depending on the app we are deploying. For this reason we have `liveness probes`
which allow us to define how we can test if our container is running properly.
You can see in the manifest below that we have defined the liveness probe within the
pod definition. We define the `httpGet` endpoint to test, which will wait for a successfull response
greater than 200 and less than 400.

- **initialDelaySeconds** How long to wait after deployment before testing.
- **timeoutSeconds** How long the request will wait for a successfull response.
- **periodSeconds** How often to run the liveness probe.
- **failureThreshold** How many failures until kubernetes restarts the container.

```yaml
# Add to your container manifest
livenessProbe:
  httpGet:
    path: /healthy
    port: 8080
  initialDelaySeconds: 5
  timeoutSeconds: 1
  periodSeconds: 10
  failureThreshold: 3
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

To apply the new changes 
```
kubectl apply -f kuard-deployment.yml
``` 

## See it in action
run `kubectl port-forward deployment/kuard-deployment 8080:8080`
open your browser to `localhost:8080`

Here you can go to the liveness probe tab and see the probes that have targeted the service.

Clicking `fail` and leaving it in that state for the time required in our above definition(in this case 30 seconds) will force kubernetes to restart the container.

You can see this by running `kubectl get pods` in a terminal window and noticing the restarts column incremented on one of the pods.

Once you are done this you can stop the port forwarding `ctrl c`

*More on [Liveness Probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-a-liveness-http-request)*

**Next step**: [Readiness Probes](06-readiness_probes.md)
