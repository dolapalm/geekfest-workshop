# Ingress 

## Use case

When you have a Kubernetes [Service](07-services.md), it is exposed to the inside 
of the Kubernetes cluster.  However, if you want to expose it to the rest of the world, 
you have no way of telling Kubernetes to do so.  This is where ingress resources comes in.

If your cluster has an Ingress controller, and you get the right to start an Ingress resource, 
you can tell Kubernetes to expose your service on specific `host` and/or `path`. 

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: api-server-ingress-rule
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: somehost.example.com
    http:
      paths:
      - path: /my/path
        backend:
          serviceName: kubernetes
          servicePort: 443
  tls:
  - hosts:
    - somehost.example.com
    secretName: my-secret
```

In this example we tell that if a requests comes in with our host `somehost.example.com` and on
path `/my/path`, to redirect it to our service `web-prod`.  However, the ingress resource might not
own our `host` in it's default certificate.  To allow it to expose a valid certificate for us, we specify 
a [secret](09-config_maps_and_secrets.md) to it containing our Certificate Key and Public Key (tls.crt and 
tls.key are important!).  Finally, we ask the ingress to rewrite the path to `/` on our micro-service.

It's important to notice that we use a nginx ingress resource in our example (annotations).  


More on `Ingress`; [https://kubernetes.io/docs/concepts/services-networking/ingress/]