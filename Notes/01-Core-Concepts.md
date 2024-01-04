# Kubernetes Commands:

* For help for commands `kubectl <command> --help`
* For getting information about the object, `kubectl explain <object>` where the object can be "Pod", "ReplicationController", "ReplicaSet". 
* We can also use shortform for these objects. It can be used as `kubectl explain rs` to explain about ReplicaSet or `kubectl edit rs rs-name` to edit the ReplicaSet.
* Get all the existing objects `kubectl get all` This can be added with the --watch `kubectl get pods --watch` to monitor for changes.
* To recreate from scratch, `kubectl replace --force -f cong.yml`. 
* To copy the YAML of existing objects, we can do `kubectl get <object> objectName -o yaml > filename.yaml`
* Objects that required `apps/v1`: 
  * ReplicaSet
  * Deyployment
  * DaemonSet
* For restarting the controlplane components, we can use the delete command cuz its static nodes & they will be brought back again. So, it can be done as `kubectl delete pod kube-apiserver-controlplace ...other_pods`

## Kubernetes Pods:
* To run a pod using a image `kubectl run podName --image imageName`
* To create it with a port, `kubectl run pod podName --image imageName --port PORT` when we want to expose on container port.
* To run a pod using a config file `kubectl create -f fileName.yml`
* Apply changes to a running config file by using `kubectl apply -f fileName.yml`
* To get the cmd as YAML file, we can use `kubectl run podName --image imageName --dry-run=client -o yaml`
* To get all the running pods `kubectl get pods` or with the wide option `kubectl get pods -o wide`
* To delete a pod `kubectl delete pod podName`
* To get information about a pod `kubectl describe pod podName`
* Basic Pod Config:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-custom
  name: nginx-custom
spec:
  containers:
  - image: nginx
    name: nginx-custom
    ports:
      - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## Kubernetes ReplicationController/ReplicaSet:
* To create a ReplicaSet (RS) `kubectl create -f rsDefinition.yml`
* To get the running ReplicaSet `kubectl get replicaset`
* To delete the ReplicaSet `kubectl delete rs rsName1 rsName2`
* Updaing the ReplicaSet can be done in two ways. ReplicaSet Config file or the ReplicaSet directly.
    * To update the ReplicaSet Config file, `kubectl replace -f rsDefinition.yml`
    * To edit the ReplicaSet directly , `kubectl edit rs rsName`
* To get information about the ReplicaSet, `kubectl describe rs rsName`
* Scaling of the RS from the config file, `kubectl replace -f rsConfig.yml`
* To Scale directly from the CMD, `kubectl scale -replicas=COUNT rs.yml` or `kubectl scale --replicas=count rs rsName`
* Sample Configuration:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  creationTimestamp: null
  labels:
    app: frontend-deployment
  name: frontend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: frontend-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

## Kubernetes Deployment:
* To create the deployemnt `kubectl create -f deployment-config.yml`
* To get the deployment `kubectl get deployments`
* To delete a deployment `kubectl delete deployment deploymentName`
* To update it, we can do it directly or the deployment-config file:
    * Directly updating the object by `kubectl edit deployment deploymentName`
    * Editing & then replacing the deployment `kubectl replace -f deployment-config.yml`
* To get information about the Deployment, `kubectl describe deployment deploymentName`
* Sample Code
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: frontend-deployment
  name: frontend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: frontend-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

## Kuberenetes Services:
* To create the service `kubectl create -f service.conf.yml`
* To get the services `kubectl get services` or `kubectl get svc`
* To delete service `kubectl delete svc svcName`
* To update:
    * Update the Service Object directly `kubectl edit svc svcName`
    * Replace it using `kubectl replace -f svConf.yml`
* Describe using `kubectl describe svc svcName`

## Kubernetes Namespaces & Resource Quota:
* To create a namespace:
    * By means of a command `kubectl create namespace namespaceName`
    * By meands of code but then we have to create it by `kubectl create -f namespace-conf.yaml`.
* Operation such as `replace`, `edit`, `delete`, `get` can be done by namespace too.
* Getting a resource in a namespace `kubectl get <object> -n namespaceName`. To get that resource in all namespace, `kubectl get <object> --all-namespace`. Or all the ns can be seen by `kubectl get pods -A`
* To create a resource in that namespace, `kubectl create -f conf.yaml -n namespaceName`
* Switch between namespace `kubectl config set-context $(kubectl config current-context) --namespace=newNamespaceName`
* To create a ResourceQuota, it can be done by: `kubectl create quota quotaName --hard=requests.cpu=4,requests.memory=5Gi,limits.cpu=10,limits.memory=10Gi` or can be done as code & created using `kubectl create -f quoat.yml`.
* Namespace can also be identified as `ns`. 
* Sample namespace:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: dev-ns
spec: {}
status: {}
```
* Sample Resource Quote:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: dev-quota
  namespace: dev-ns
spec: 
  hard:
    requests.cpu: 4
    requests.memory: 3Gi
    limits.cpu: 5
    limits.memory: 5Gi

status: {}
```

## Dry Run YAML File creation commands:
* To get a YAML file for Pod `kubectl run podName --image imageName --dry-run=client -o yaml > pod-config.yaml`
* To get the YAML file for Deploment `kubectl create deployment deploymentName --image imageName --dry-run=client -o yaml > deployment-config.yaml`
* To create YAML file for Service, we can use similar command while provding the service type like `kubectl create service serviceType serviceName --tcp=port:targetPort --dry-run=client -o yaml > service-conf.yaml`. Example would be `kubectl create service clusterip frontend-clusterip --tcp=80:8080 --dry-run=client -o yaml >service-conf.yml`
* To create the namespace `kubectl create namespace nName --dry-run=client -o yaml > namespace-conf.yml`
* To create ResourceQuota `kubectl create quota quotaName --dry-run=client -o yaml --hard=key=val,key=val`. 

## Declarative Method: (better way)
* To apply the config file, `kubectl apply -f cong.yaml` if it doenst exists it will create else it will update.
* To give a directory of files, `kubectl apply -k directory`. 
* To expose a pod which will automatically create a Service for it, we can use `kubectl expose pod podName --port=PORT --target-port=PORT --type=NodePort|ClusterIP --name SERVICENAME`.