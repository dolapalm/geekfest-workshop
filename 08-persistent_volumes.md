# Persistent Volumes
When running a pod on kubernetes, any pod that gets restarted or deleted will lose all data that was associated with it. This is usually a good thing as we want a clean state whenever we restart a container in order to have consistent results. However there are some cases where we want to maintain the state of a pod and for this reason kubernetes also provides a way to persist data using volumes.

To enable a persitent volume, we first define the volume and its `hostPath`, then we define the `volumeMount` where the container will mount the volume defined.

```yaml
# Add the following to the spec right above the container definition 
volumes:
  - name: "kuard-data"
    hostPath:
      path: "/home/docker"


# Add the following to your container manifest
volumeMounts:
  - mountPath: "/data"
    name: "kuard-data"
```

*kuard-deployment.yml*
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuard-deployment
  labels:
    app: kuard
  annotations:
    source: https://www.github.com/me/myproject.git
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
      volumes:
        - name: "kuard-data"
          hostPath:
            path: "/home/docker"
      containers:
        - image: gcr.io/kuar-demo/kuard-amd64:1
          name: kuard
          ports:
            - containerPort: 8080
              name: my-port
              protocol: TCP
          volumeMounts:
            - mountPath: "/data"
              name: "kuard-data"
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
          ports:
            - containerPort: 8080
              name: my-port
              protocol: TCP
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
To apply the new changes 
```
kubectl apply -f kuard-deployment.yml
``` 

## See it in action
Lets test the volume mount by adding a file into the volume and then see that it can be found within the container.
- Create a file in our minikube VM 
```
minikube ssh sudo touch /home/docker/test
```
- Open the demo in the browser and navigate to `file system browser` then navigate to `/data` and you should see the `test` file.
- Delete the current deployment
```
kubectl delete deployment kuard-deployment
```
- Redeploy the deployment
```
kubectl apply -f kuard-deployment.yml
```
- Retry the file system browser in your browser and you should see that the file still exists.


*More on [Persisting Data](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)*

**Next step**: [Config Maps and Secrets](09-config_maps_and_secrets.md)
