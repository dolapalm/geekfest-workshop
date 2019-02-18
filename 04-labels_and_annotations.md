# Labels and annotations

## Labels
Labels are a way to describe your resources. They won't impact how your application will behave but will allow you to apply filter when you want to act on a specific group.

A label is a key/value pair that can be applied to resources.

More on labels: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/

```yaml
# Add to your deployment resource
spec:
  template:
    metadata:
      labels:
        app: kuard
        env: prod
        ver: "1"
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

To apply your new changes:
```bash
kubectl apply -f kuard-deployment.yml
```

### See it in action
To view how labels work, we'll create a few deployments.
```bash
kubectl run kuard-staging --image=gcr.io/kuar-demo/kuard-amd64:2 --replicas=1 --labels="env=staging,app=kuard,ver=2" 
kubectl run db-prod --image=gcr.io/kuar-demo/kuard-amd64:1 --replicas=2 --labels="env=prod,app=db,ver=1" 
kubectl run db-staging --image=gcr.io/kuar-demo/kuard-amd64:2 --replicas=2 --labels="env=staging,app=db,ver=2" 
```

### Using label selectors
Given the information above, the following commands would return a different list:
```bash
kubectl get deployments --show-labels

NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       LABELS
db-prod            2         2         2            2           20s       app=db,env=prod,ver=1
db-staging         2         2         2            2           14s       app=db,env=staging,ver=2
kuard-deployment   3         3         3            3           1m        app=kuard
kuard-staging      1         1         1            1           34s       app=kuard,env=staging,ver=2

kubectl get pods --selector="env=prod"

# Returns pods for kuard-prod and db-prod
NAME                              READY     STATUS    RESTARTS   AGE
db-prod-6797f88d5f-k6cp7          1/1       Running   0          52s
db-prod-6797f88d5f-w7skk          1/1       Running   0          52s
kuard-deployment-8f449db6-6wkjv   1/1       Running   0          1m
kuard-deployment-8f449db6-dgfc9   1/1       Running   0          1m
kuard-deployment-8f449db6-zgzt4   1/1       Running   0          1m

kubectl get pods --selector="env=staging,app=db"

# Only returns pods for db-staging
NAME                          READY     STATUS    RESTARTS   AGE
db-staging-7ddb9cbb89-45lpw   1/1       Running   0          2m
db-staging-7ddb9cbb89-n9wx6   1/1       Running   0          2m

kubectl get pods --selector="app in (db, kuard)"

# Would return all pods since they all match the labels
kubectl get pods --selector="app in (db, kuard)"
NAME                              READY     STATUS    RESTARTS   AGE
db-prod-6797f88d5f-k6cp7          1/1       Running   0          2m
db-prod-6797f88d5f-w7skk          1/1       Running   0          2m
db-staging-7ddb9cbb89-45lpw       1/1       Running   0          2m
db-staging-7ddb9cbb89-n9wx6       1/1       Running   0          2m
kuard-deployment-8f449db6-6wkjv   1/1       Running   0          3m
kuard-deployment-8f449db6-dgfc9   1/1       Running   0          3m
kuard-deployment-8f449db6-zgzt4   1/1       Running   0          3m
kuard-staging-8c4b8f66f-zbvsr     1/1       Running   0          2m

kubectl get pods --selector="canary"

# Would return no pod since they don't have a canary label
No resources found.
```

### Cleaning up
Let's remove these extra deployments.
```bash
kubectl delete deployment kuard-staging
kubectl delete deployment db-prod
kubectl delete deployment db-staging
```

## Annotations
Annotations are metadata that represent arbitrary non-identifying information to a ressource so that they can be used by tools and libraries.

Here are some example of metadata:
- Build, release, or image information like timestamps, release IDs, git branch, PR numbers, image hashes, and registry address.
- Pointers to logging, monitoring, analytics, or audit repositories.
- Client library or tool information that can be used for debugging purposes: for example, name, version, and build information.
- User or tool/system provenance information, such as URLs of related objects from other ecosystem components.
- Lightweight rollout tool metadata: for example, config or checkpoints.
- Phone or pager numbers of persons responsible, or directory entries that specify where that information can be found, such as a team web site.
- Directives from the end-user to the implementations to modify behavior or engage non-standard features.

A good example is the nginx resource whose behaviour can be configured through annotations. For example, the following annotation would allow nginx to overwrite the URI that is presented to the user:
```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /
```

Instead of accessing the service through a URL like https://myservice.domain.com/mydeployment/, the annotation would simplify it to https://myservice.domain.com

More on annotations: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/

More on nginx annotations: https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md

### Adding an annotation through manifest

An annotation is added in the metadata section of the manifest:
```yaml
metadata:
  annotations:
    source: https://www.github.com/me/myproject.git
```

**Next step**: [Liveness Probes](05-liveness_probes.md)
