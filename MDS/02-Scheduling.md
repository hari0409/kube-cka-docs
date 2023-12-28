# Kubernetes Scheduling:

## Manual Scheduling:
* This needs to be done when the scheduler is not present. This can be done specifying the `nodeName` property in the pod-conf.yaml file under the `spec`
* If we want to migrate pods, we can create a binding object & this object can be sent as a POST request to the Node. The Request is given by:
```bash
curl --header "Content-Type: application/json" --request POST --data BINDING_JSON_CONVERTED_FROM_YAML http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding 
```
* Get nodes existing in the implemetation using the command `kubectl get nodes`

## Labels & Selectors:
* Labels can be added to any object in the metadata field under the filed `labels`.
* These labels can be matched by means of the selector which can be used ReplicaSet, Deployment etc. But we can also extract nodes based on these label from CLI by `kubectl get nodes --selector=key:val,key:val`

## Taints & Tolerations:
* Taint &rarr; Nodes
* Tolerations &rarr; Pods.
* Taints can be added to the node in two ways:
    * Modification of the config file of the node
    * Adding through CLI `kubectl taint node nodeName key=value:effect` where effect can be:
        * NoSchedule
        * NoExecute
        * PreferNoSchedule
* Toleration can be added to the pods by adding the `tolerations` as an array of objects which contains `key`,`value`,`operator`,`effect` which will be
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations:
    - key: "app"
      value: "blue"
      operator: "Equal"
      effect: "NoSchedule"
```
* When no value need to be specified, then we have to use `Exists` as the value for the toleration.
To remove a taint from the node, we can use the same command but with a minus (-) like `kubectl taint node nodeName key=value:effect-`

## Node Selector
* This is a simple low level implementation of creatin affinity between the node & pod by means of nodeSelector. 
* Add the label using `kubectl label node nodeName key=value`
* Add the code 
```yaml
apiVersion: v1 
kind: Pod
metadata: 
    name: nginx-app
    labels: 
        tier: frontend
spec:
    containers: 
    - name: nginx-app
        image: nginx
    nodeSelector: # This is used for identifying the node
        size: Large
```

## Node Affinity
* This is complex implemention of node selector but provides more flexibility. This can be done by:
```yaml
affinity: 
    nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions: 
                - key: key 
                  operator: In | NotIn | Exists | DoesNotExist
                  value:
                  - val2
```
in the template of the Deployment or of pod config inline to the containers. 
* Thus the whole code becomes:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: red
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      app: red
  template:
    metadata:
      labels:
        app: red
    spec:
      containers:
      - image: nginx
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: In | NotIn | Exists | DoesNotExist
                value:
                - val1
                - val2
```

## Resource Requirement & Limits:
* Resource limitation can be done by Container, Pod & Node.
* For Containers &rarr; Restriction & Limit, Pods &rarr; LimitRange, Node &rarr; ResourceQuota.
* Creating a Restriction & limit for container:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
      resources:
        requests:
          cpu: 10
          memory: "4Gi"
        limits:
          cpu: 13
          memory: "6Gi"
```
* For a Pod using LimitRange which will be deployed in the namespace level.
```yaml
apiVersion: v1
kind: LimitRange
metadata: 
  name: cpu-LimitRange
spec:
  limits:
    - default: # limit
        cpu: 1
      defaultRequest: # requests
        cpu: 2
      max: # Limit for container
        cpu: 1
      min: # Request for container
        cpu: 100m
      type: Container
```
* For node, we can use ResourceQuoate which is already covered. 

## DaemonSet:
* This can be used to run a pod in every node of the cluster. This is very similar to the Relicaset (Deployment) except it deoesnt have `statergy` & `replicas`.
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - image: registry.k8s.io/fluentd-elasticsearch:1.20
        name: fluentd-elasticsearch
```

## Static Pods:
* This can be used for creating pods in the node when there is no masternode & there is only workernodes.
* The pod configuration files can be places in a path configured to be the `staticPodPath` which will be configued in the kubelet configuration.
* This path most probably will be `/etc/kubernetes/manifest`.
* If need to find the path, `/var/lib/kubelet` tp get the config file to get the `statisPodPath`.

## Custom Scheduler:
* This can be used to configre our own algorithms or plugins. This can be done by creating a Scheduler Config File:
```yaml
apiVersion: kubescheduler.config.k8s.io/v1beta2
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler # Name of the scheduler profile
leaderElection:
  leaderElect: true
  resourceNamespace: kube-system
  resourceName: lock-object-custom-scheduler
```
* Then we can create a Pod Config file which will be implementing this scheudler:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
    - name: my-scheduler
      image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
      commands:
        - kube-scheduler
        - --address=127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --config=/etc/kubernetes/custom-scheduler.conf

```
* Then we can use this scheduler in any object by specifying `schedulerName` field in the spec like
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-custom
  name: nginx-custom
spec:
  containers:
  - image: nginx
    name: nginx-custom
    ports:
      - containerPort: 8080
  schedulerName: custome-scheduler
```

## Multiple Profile in Schduler:
* Instead of deploying multiple scheduler, we can deploy multiple profiles of the schedulers. This can be done by specifying the schedulerName & its extensionpoint with the plugins requried.
```yaml
apiVersion: kubescheduler.config.k8s.io/vl
kind: KubeSchedulerConfiguration 
profiles:
  - schedulerName: my-scheduler-1
    plugins: 
      score:
        disabled:
          - name: TaintToleration
        enabled: 
          - name: MyCustomPluginA
          - name: MyCustomPluginB
  - schedulerName: my-scheduler-2
    plugins: 
      prescore:
        disabled:
          - name: "*"
      score:
        disabled:
          - name: "*"

```
* This can be then deployed in a Pod or Deployment & can be used in any Kube Object.
