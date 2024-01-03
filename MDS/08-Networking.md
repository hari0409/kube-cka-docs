# Networking in K8s:

## Getting Information:
* This can be obtained by nslookup command `nslookup -nlpt | grep server-name`.
* Get inforamtion about the services using the command `ps -aux | grep service-name`
* Use the **`logs/describe/yaml`** for getting the information abt the deployed objects. 
* Get the iptables of the node `iptable -L -t nat | grep service-name`.
* Get the interface for a particular type of connection `ip address show type bridge`.
* To get iprange:

| Data                      | Source                                                                                         | 
| ------------------------- |:-------------------------------------------------------------------------------------------:   | 
| IP Range of Pods          | Config from the kube-proxy pod                                                                 |
| IP Range of Services      | Fromt the manifest file of kube-apiserver                                                      |  
| IP Range of Nodes         |  Check the IP from `kubectl get nodes -o wide `& then the `ip address `to check the range      |   

## Getting CNI Configration:
* Get all the available configuration in the folder `/etc/cni/bin` & the network configuration will be in the folder `/etc/cni.net.d`.
* The configruation of the `ipam` is present in the network configuration file which can be `host-local` or `DHCP`. 
```yaml
{
    "cniVersion": "0.2.0",
    "name": "mynet",
    "type": "net-script",
    "bridge": "cni0",
    "isGateway": true
    "isMsaq": true,
    "ipam": {
        "type": "host-local" | "DHCP",
        "subnet": "10.244.0.0/16",
        "routes": [
            {
                "dst": "0.0.0.0/0"
            }
        ]
    }
}
```

## Pod Networking:
* Here there will be pods on a node. They will be connected by means of the bridge & virtual eth cables. 
* The connection between the pods in various nodes will be done by the router configured in the k8s cluster which will be containing the default gateway for each node. 
* All of these networking can be done by networking plugin developed following the CNI standard which can then be deployed as a pod in the cluster. 

## Service Networking:
* Services include `ClusterIP` & `NodePort`. ClusterIP is used for communication inside the cluster & the NodePort is used for communication with the external environment.
* This is configured by the kube-proxy. 

## DNS:
* The DNS name will be **created for services** only but the Corefile can be configure to create domain name for pods too.
* The URL for accessing service can be:
```
http://service-name
http://service-name.namespace
http://service-name.namespace.svc
http://service-name.namespace.svc.cluster.local
``` 
* The URL for accesing the pod will be `IP-DASHED.namespace.svc.cluster.local`.
* Components of CoreDNS
    * Pods deployed as `ReplicaSet`
    * Access the ReplicaSet by means of a `ClusterIP Service`
    * Store the configuration as a `ConfigMap`
* The configuration for the Configfile will be containing the `root domain name` like
```conf
.:53 {
    errors
    health
    kuberbetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure # for creating domain for pod
        upstream
        fallthrough in-addr.arpa ip6.arpa
    }
    prometheus :9153
    proxy . /etc/resolv.conf
    cache 30
    reload
}
```
* The DNS configuration of a server(pod & CoreDNS) is present in the `/etc/resolv.conf` file of the pods which will be done automatically by k8s. In case of pods, it will be containing the IP of the DNS service used to access the DNS. In case of the DNS server, it will contain all the IP-NAME mapping.

## Ingress:
* This is used as a proxy for the kubernetes. This contains two part.
    * Ingress Controller (The proxy which will be deployed in the cluster)
    * Ingress Resource (The rules which will be deployed in hte Controller to route to different services based on those rules).
* The process of creating a Ingress Controller is given [here](../Complete-Ingress-Files/).
* The Ingress Resources are used for implementing the rules where the fields required are:
    * host [array]
    * paths
        * path
        * pathType
        * backend
            * service
                * name
            * port
                * number
* We can also include some annotatino for giving additional proerpty for the nginx server.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  creationTimestamp: null
  name: nginx-ingress
  namespace: app-space
  annotations:  
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Exact
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /watch
        pathType: Exact
    - host: domain.com
      http:
        paths:
        - backend:
            service:
                name: wear-service
                port:
                number: 8080
            path: /wear
            pathType: Exact
        - backend:
            service:
                name: video-service
                port:
                number: 8080
            path: /watch
            pathType: Exact
status:
  loadBalancer: {}
```
