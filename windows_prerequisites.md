# Windows installation 
## Requirements
- Have VTX enabled in your BIOS (must be done by your Technical Service Desk) (See instructions below to determine if it is already enabled or not)
- Have administrative rights to install the below softwares (must be done by your Technical Service Desk if you don’t have the rights)
 
## Software installation
- VirtualBox: Follow instructions on https://www.virtualbox.org/wiki/Downloads
- Minikube: Download minikube-installer.exe from this page and follow the instructions: https://github.com/kubernetes/minikube/releases/tag/v0.33.1
- Kubectl: Download the following binary: https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/windows/amd64/kubectl.exe
  - Save it to your minikube installation folder: C:\Program Files\Minikube
 
How to know if you have VTX enabled?
- After installing VirtualBox, launch it and click on the “New” to create a new virtual machine.
- In the list of versions, if you can select either 32-bit or 64bit, VTX is enabled.
- If you only have 32-bit, VTX is disabled. You need to have it enabled by your Technical Service Desk.
 
All the following steps will be done through Windows Command prompt (cmd)
 
## Execute Minikube the first time
It is recommended to run the configuration in an environment without proxy. Otherwise, Minikube may behave differently when you move from an environment with proxy to one without.
```bash
unset http_proxy https_proxy noproxy HTTP_PROXY HTTPS_PROXY NOPROXY
minikube start
```

## Test minikube and kubectl
```bash
kubectl get pods -n kube-system
$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
deployment.apps/hello-minikube created
$ kubectl expose deployment hello-minikube --type=NodePort
service/hello-minikube exposed
 
# We have now launched an echoserver pod but we have to wait until the pod is up before curling/accessing it
# via the exposed service.
# To check whether the pod is up and running we can use the following:
$ kubectl get pod
NAME                              READY     STATUS              RESTARTS   AGE
hello-minikube-3383150820-vctvh   0/1       ContainerCreating   0          3s
# We can see that the pod is still being created from the ContainerCreating status
$ kubectl get pod
NAME                              READY     STATUS    RESTARTS   AGE
hello-minikube-3383150820-vctvh   1/1       Running   0          13s
# We can see that the pod is now Running and we will now be able to curl it:
$ minikube service hello-minikube --url
 
 
Hostname: hello-minikube-7c77b68cff-8wdzq
 
Pod Information:
                -no pod information available-
 
Server values:
                server_version=nginx: 1.13.3 - lua: 10008
 
Request Information:
                client_address=172.17.0.1
                method=GET
                real path=/
                query=
                request_version=1.1
                request_scheme=http
                request_uri=http://192.168.99.100:8080/
 
Request Headers:
                accept=*/*
                host=192.168.99.100:30674
                user-agent=curl/7.47.0
 
Request Body:
                -no body in request-
 
 
$ kubectl delete services hello-minikube
service "hello-minikube" deleted
$ kubectl delete deployment hello-minikube
deployment.extensions "hello-minikube" deleted
$ minikube stop
Stopping local Kubernetes cluster...
Stopping "minikube"...
```
