# Deployments
Deployments exist in order for us to manage the release of new versions of our applications by the use of the `rollout`.
Using deployments we can safely and reliably roll out new versions of our application without any downtime.
In kubernetes, replica sets manage pods and deployments manage replica sets where replica sets define how many replicas of a pod should be running in the cluster. Therefore anything you define in the deployment will be the desired state for all resources it manages.

NOTE: If you try to modify a pod or replica set that was created by a deployment it will be reset by the desired state of the deployment. Only by modifying the deployment can you make a change to any of the underlying elements.

Before creating a deployment, delete your current running pod as we will now redeploy it using a deployment.

## Create a deployment
Modify your pod manifest from the previous step to turn it into a deployment.

*Notice the spec.containers section is essentially what remains from the pod definiton we had before.*
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
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
```
```
kubectl apply -f kuard-deployment.yaml
```


## See it in action
```
kubectl get all
```
You will see an output similar to the following:
```
NAME                                    READY     STATUS    RESTARTS   AGE
pod/kuard-deployment-6b6995ff77-846bq   1/1       Running   0          5s
pod/kuard-deployment-6b6995ff77-fhplk   1/1       Running   0          5s
pod/kuard-deployment-6b6995ff77-fs67k   1/1       Running   0          5s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   14d

NAME                               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kuard-deployment   3         3         3            3           5s

NAME                                          DESIRED   CURRENT   READY     AGE
replicaset.apps/kuard-deployment-6b6995ff77   3         3         3         5s
```


Now lets try and update the container that is running to observe the rolling update.

```
kubectl edit deployment kuard-deployment
```
Then hit `i` to edit the section of the file that says:
`- image: gcr.io/kuar-demo/kuard-amd64:1` to `- image: gcr.io/kuar-demo/kuard-amd64:2`

This will change the version running and we can observe how kubernetes handles it.
you can then save and exit the editor by hitting `ESC` then `:wq` `enter`.

```
kubectl get all
```

If you did it fast enough you should see something along the lines of the following.
```
NAME                                    READY     STATUS              RESTARTS   AGE
pod/kuard-deployment-6b6995ff77-846bq   1/1       Running             0          2m
pod/kuard-deployment-6b6995ff77-fhplk   1/1       Running             0          2m
pod/kuard-deployment-bc545cd45-9jxts    0/1       ContainerCreating   0          3s
pod/kuard-deployment-bc545cd45-mpsrh    0/1       ContainerCreating   0          3s
```
We can see that it has killed one of the pods and has started to create two new ones with the new version while two of the
other pods remained up during this time ensuring no downtime.
If you continue to run `kubectl get all` you will see it terminate the other pods and create new ones and finaly stabalize at 3 running pods.

What we saw here was that the `maxSurge` set to 1 allowed us to temporarily have one extra during creation time and the `maxUnavailable` allowed only 1 of the 3 pods to be unavailable during the rollout, leaving us with a minimum of 2 running at all times.

*More on [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)*

**Next step**: [Requests and Limits](03-requests_and_limits.md)
