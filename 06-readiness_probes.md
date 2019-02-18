# Readiness Probe
In the previous section we saw the liveness probe which tells kubernetes if the app running is in
a healthy state and whether it needs to be restarted. Along with the liveness probe we also have the readiness
probe which lets kubernetes know whether or not it should send traffic to that container. A good example of this is
during startup, we do not want to send traffic to a container that has started but may still be initializing its application
logic.

- **initialDelaySeconds** How long to wait after deployment before testing.
- **periodSeconds** How often to run the liveness probe.
- **failureThreshold** How many failures until kubernetes determines that the container is not ready.
- **successThreshold** How many successfull probes to be considered ready.

```yaml
# Add to your container resource
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 2
  initialDelaySeconds: 0
  failureThreshold: 3
  successThreshold: 1
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
```

To apply the new changes 
```
kubectl apply -f kuard-deployment.yml
``` 

## See it in action
Run `kubectl port-forward deployment/kuard-deployment 8080:8080`.

Open your browser to `localhost:8080`.

Here you can go to the Readiness probe tab and see the probes that have targeted the service. Clicking `fail` and leaving it in that state for a while you will be able to show us a different reaction.

You can see this by running `kubectl get pods` in another terminal window  and noticing that the restarts column is not changing. However kubernetes has not listed the container as ready as we see by the 0/1 in the ready column. This will inform any load balancers not to direct traffic to this pod.

Once you are done with this exercise you can click `succeed` in the browser readiness tab to put the pod back in a healthy state. You can then run `kubectl get pods` and see that it has returned to 1/1 in the READY column.

You can now cancel the port-forward `ctrl c`.

*More on [Readiness Probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-readiness-probes)*

**Next step**: [Services](07-services.md)
