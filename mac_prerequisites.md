# MacOS installation of minikube

*For the sake of this workshop please work on a `public network` without any `proxy` variables enabled.
The use of a VPN will also cause issues that are out of the scope of this workshop.*

*Be sure to also unset any terminal environment variables related to proxies if you have any.*

ex: `unset http_proxy https_proxy noproxy HTTP_PROXY HTTPS_PROXY NOPROXY`

## Prerequisites
- VirtualBox: Follow instructions on https://www.virtualbox.org/wiki/Downloads
- run `brew cask install minikube` in a terminal window
- To see if it installed correctly run `kubectl version` in a terminal.\
	It should show something along the lines of the following:
```
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.11", GitCommit:"637c7e288581ee40ab4ca210618a89a555b6e7e9", GitTreeState:"clean", BuildDate:"2018-11-26T14:38:32Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"darwin/amd64"}
Unable to connect to the server: net/http: TLS handshake timeout
```
The last line is normal as we have not started the cluster yet.

## To start Minikube for the first time
Run `minikube start` in a terminal window (with no proxy variables set)

This will take a few minutes to start up

```
Starting local Kubernetes v1.13.2 cluster...
Starting VM...
Downloading Minikube ISO
 181.48 MB / 181.48 MB [============================================] 100.00% 0s
Getting VM IP address...
Moving files into cluster...
Downloading kubeadm v1.13.2
Downloading kubelet v1.13.2
Finished Downloading kubeadm v1.13.2
Finished Downloading kubelet v1.13.2
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Stopping extra container runtimes...
Starting cluster components...
Verifying kubelet health ...
Verifying apiserver health ...
Kubectl is now configured to use the cluster.
Loading cached images from config file.


Everything looks great. Please enjoy minikube!
```

Run `kubectl cluster-info`and you should see something along the following:
```
Kubernetes master is running at https://192.168.99.144:8443
KubeDNS is running at https://192.168.99.144:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```


## Troubleshooting
If your minikube start fails at any point its usually easier to do a `minikube delete`,
Restart the Virutalbox GUI and go into network under tools and delete the network for 192.168.99.1/24 and then retry starting minikube.
