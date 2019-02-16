# ConfigMap and Secrets

## ConfigMap

ConfigMap are made to store (*drumroll*) configurations!  
It's a Kubernetes resource made to store a "key:value" pair.
A good example of a configuration are environment variables.

Add a new config map resource to the manifest;

*kuard-deployment.yml*
```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
    another-param: another-value
    extra-param: extra-value
    my-config.txt: |
      # This is a sample config file that I might use to configure an application
      parameter1 = value1
      parameter2 = value2
  
```

```
kubectl apply -f kuard-deployment.yml
```

We will now tell our pods how to access this config map.

ConfigMap can be referenced into Pods in 2 ways;
1. Environment variables
    ```yaml
   # Add the following to your container manifest
    env:
     - name: ANOTHER_PARAM
       valueFrom:
         configMapKeyRef:
           name: my-config
           key: another-param
    ```
2. Volume (in which the key becomes a file name and the value it's content)

    ```yaml
    # Add the following to the volume definition of the pod
    - name: config-volume
      configMap:
        name: my-config 
    
    
    # Add the following to your container volumemounts manifest
    - name: config-volume
      mountPath: /config
    ```


```yaml
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
            path: "/var/lib/kuard"
        - name: config-volume
          configMap:
            name: my-config 
      containers:
        - image: gcr.io/kuar-demo/kuard-amd64:1
          name: kuard
          env:
            - name: ANOTHER_PARAM
              valueFrom:
                configMapKeyRef:
                  name: my-config
                  key: another-param
          volumeMounts:
            - mountPath: "/data"
              name: "kuard-data"
            - name: config-volume
              mountPath: /config
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
  
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
    another-param: another-value
    extra-param: extra-value
    my-config.txt: |
      # This is a sample config file that I might use to configure an application
      parameter1 = value1
      parameter2 = value2

```
To apply the new changes 
```
kubectl apply -f kuard-deployment.yml
``` 

## See it in action
Lets test the ConfigMap mount and see that it can be found within the container.
- Open the demo in the browser and navigate to `file system browser` then navigate to `/config` and you should see 3 files;
    - another-param
    - extra-param
    - my-config.txt
- Open the demo in the browser and navigate to `Environment`.  There should be an environment variable `ANOTHER_PARAM` and it's value `another-value`.


More on ConfigMap: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

## Secrets

Secrets are similar to configmap but were designed to keep it secret. 
They are useful for putting certificate public and private key. 

**Even though the name and feature call this a secret, it's more a base64 encoded string.
The way Kubernetes manage secrets might not be what you would expect from a secret.  Please make sure 
to use an add-on encryption providers**


```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMyekNDQWNPZ0F3SUJBZ0lKQUkyQXhWem5QUU5OTUEwR0NTcUdTSWIzRFFFQkN3VUFNQnd4R2pBWUJnTlYKQkFNTUVXdDFZWEprTG1WNFlXMXdiR1V1WTI5dE1CNFhEVEUzTURNeE9USXdNVFF5TVZvWERURTRNRE14T1RJdwpNVFF5TVZvd0hERWFNQmdHQTFVRUF3d1JhM1ZoY21RdVpYaGhiWEJzWlM1amIyMHdnZ0VpTUEwR0NTcUdTSWIzCkRRRUJBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRRE82SGplVmMvT3pDcUNLVGI5RVNCNGZPdUh5d001UjkzcTNzc2wKNXVGeTZzb29PdlJmcFExQURMWmpOd2ZOYUVrVmNORXNzSFdDSCtEaGJ2c3RhOTB6VFhZZFBGV3ZWYVhjMzJ1eQpkUURSK0ZJUnhsNWMxb0hyaEQ2eVlKSm9wOU55dzBod3JmT3lqNytOVVcwZmdTYVZ0YmRyTGxoUVAwVnVvUVVHClJsSHBsN2ltcVAzUGxnUUxtbzh4bk5RMStSMDcybDByQi9CcVVHZEc2TUE2UlhmOU5peGFFQ05WZ1N4dXUrQkIKd3gxdXBMLzZjMHJKYXFqTHBjRXFveXA1RkdvOHR0T3ZXcXdEdWtNQVRKRC83RWk1TXpTNlJkUWpGQzE5cmg4OAoxenBRNWhCUHhjeTNMa2ovWGFmNmVoWDVuclBjdVJ4NHZhZEdhZ0xucnhUZ2prN1hBZ01CQUFHaklEQWVNQndHCkExVWRFUVFWTUJPQ0VXdDFZWEprTG1WNFlXMXdiR1V1WTI5dE1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQ2IKdk9hNnBDTEp3OUsyLzkyOVdzbjZDRXVaclpmaFhVck5HVDg3Y1ZpYlFTajJhNDhzTWRibElqeGJ2a0FIdG1iZwptTXhNT01nZWExaEN3WkNhSnczRUNFbUNCNExIbEJUbkZEV2JkcW5SdnMrL1VpTGhwYXE0eDVqMnNwZjRWeXNZCjFYcWdrZEhJK0pRMUlJMDdwb3FCL0xrbXBCUHkzcC92Q25IbDVxZ1o1U2hTM0dDWkY0NWxIck1yd21xOHVqSmMKbkNhMENDWnBqellScDhwVDVXLzg4T0wrdG9lYjlyTjNja0ttQnR3czVxN0ozRGJma212MXNDUHdTdXNCZkFUWgp6NHNLZTRtb3BCTXdFNXlKemNqb1BlaVk2ZGhNWjJGc21PRTliNFhhTWtEUFZJemVaUzNTLzBuVUQzNnZDV3NQCmprRkpTOVk2dXFJUkY3VHA3K3EzCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBenVoNDNsWFB6c3dxZ2lrMi9SRWdlSHpyaDhzRE9VZmQ2dDdMSmViaGN1cktLRHIwClg2VU5RQXkyWXpjSHpXaEpGWERSTExCMWdoL2c0Vzc3TFd2ZE0wMTJIVHhWcjFXbDNOOXJzblVBMGZoU0VjWmUKWE5hQjY0UStzbUNTYUtmVGNzTkljSzN6c28rL2pWRnRINEVtbGJXM2F5NVlVRDlGYnFFRkJrWlI2WmU0cHFqOQp6NVlFQzVxUE1aelVOZmtkTzlwZEt3ZndhbEJuUnVqQU9rVjMvVFlzV2hBalZZRXNicnZnUWNNZGJxUy8rbk5LCnlXcW95NlhCS3FNcWVSUnFQTGJUcjFxc0E3cERBRXlRLyt4SXVUTTB1a1hVSXhRdGZhNGZQTmM2VU9ZUVQ4WE0KdHk1SS8xMm4rbm9WK1o2ejNMa2NlTDJuUm1vQzU2OFU0STVPMXdJREFRQUJBb0lCQUhiOFNVWFNvMGFSTW9EWApvcisxY2E3WVo3b1hqU3NMb1JySU5Kci9RdmNLL21aVVFPUWZ6cGJldUtRbHFWNytjY2phcisrN0tsaENiTmczCk1rclVsTWhENjRDMGliSGkxeGRGaEhHRHg0ejMrSG93VVdPaUYrU1FrRjJVRzU0RHBSMkNIODVzdXBBZENsTUMKV0hhZGxzclJUVUZkelh1WVp4MVBpOHduOUVNWDdHdW1xOG5mK2N5VENuYTVRb0d1MDBSeEZWbUpqTzJQTjVZSQpGQzRpUVJGOUQ4Vlp2QVk3OTlaL2swbm9VWk5oVXFEYUZINFV3OWQ2K2FyeXovWUJhaUhJYTVENjRTUDhTaDNqCnhYTndWZkRWNkd5LzRibnlKK24vRVlvWnZnM2ZpczdDNzR0NmM5NEIvZFJzdmpvMi9jUDNyb1JkTUFwZjdxTXYKRkxKWVZ3RUNnWUVBNW4vcFlUVlNtc0Q0dFNGdlplSEc3bzExek8zVGlYV3J6TndkbEY3VnRpSUR1SkJSMWZXbApCd2VVQUJ6Y2RURnFDNUh5TlpXV2RWSDhHR3NxVmh1dXBCSVNjUFl4b3B2ZjdFWHdBemhhN1JmQzB4MGg0T1czCmxWUGE5SDMvd2RCUjZvUnFCeE85cTBmWG9pZFVsTTQ4VWwwb2lYQjJ5UjdxbmxrM3dLQzlGd2NDZ1lFQTVjeHAKczl0V2taVGlKTVViZlhoUzNLaWFaekdqTnJUN1M5dmhIQ292NlgvcXhLR0ZWL2ljdmswdlR1RithRjBOcTIxYwo2T25jeGh6TlduVHdyNUlMWjN1bHAwckFDU1JGTDVaMDREaFhHRmsvaFRCbTluemNIcTdwbzkwSWJrbnRscEF5ClVraVJmeXlXb0FBdGVZTzFDSk9VWElBOGcxcUdIOHRLQW5UZnhiRUNnWUVBa0xpbkUzMmpTNzcxYU9TQlNQcWwKS0lwdytDWXF0eGZHc20xUnRTS0dGRUR2RFNhdit5S3NadWwvSjBMM3VDMDZZK0ZTcmdvcDJhZU1ITmpNVUJ3NQpYcEpxT2JxYUYzcSs4VjVILy8yV09WNjkyRWRtU2dweFpiU3N1TzJUYzJFVXphWXQzQnVzN3FuQTNmTEx6RkpnCjFXWGdXY3JmQ2cvN1IwakZGSkRYcUdrQ2dZQTN0YWFxZzdJbytQOGFDdTd2TEF4cWtqVmNieHd5VnczVkJpazgKdXIyQ3poQU1PMXdvUjQwNFZWM3lzWmdEbFF1TFU2Z0NqeStHbDlUdzZRaXdoNmRjSHcyNTBOVmRZZjJqMjhYWgoxYzdIaUZ0dDNwNFhnNDJab3EzaG0rUS9XSXpRdzRSdmprZzNuSWVub21OajRob3hTaFhkbHZrVW53MkZCN09aCmhOdjdBUUtCZ1FETlpHMGMvSTRldytpVkVtM0k1Y2hlRVY2L2w5dytyUHNmYW9qVTB0L0Y3RmZFdmxIanloWEMKL3BJZEsrblp0UjdBaE1iVW5LWUFBcVlhdVU0VFQ3UmRFYWdLaTRNc2hMcE1IUVBSMEYxNWp4N3RrR0FEbDRFNgpHSmNZWGEwT1VtZ2FQZWx3Ykgzb1IyeUVpSFN6Nmd1SHQ5QmxZaTJiRm1BZXpRSXc4eGRLMEE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

While the example above is nice for looking at the data (base 64 encoded), it's annoying when the data are in separate files for the automation.
You would have to take a file, base64 it's content and in-line replace in another file and repeat for each parameters

The example below would do the same as applying the manifest above but from the CLI.

```
kubectl create secret generic my-secret --from-file=kuard.crt --from-file=kuard.key
```

More on `Secrets`: https://kubernetes.io/docs/concepts/configuration/secret/
