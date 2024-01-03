# Storage in K8s

## Simple Volumes
* This can be used to mount a volume to a pod that is specified within a pod config file.
* This is done by creating a volume in the pod def file in a path of the host & then mounting it to the pod path.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx-server
      image: nginx
      volumeMounts:
        - mountPath: /data
          name: data-volume
  volumes:
    - name: data-volume
      hostPath:
        path: /data
        type: Directory
```
* But here the folder will be varying from node to node. So as to sync it, we can use volume povideres like AWS. 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx-server
      image: nginx
      volumeMounts:
        - mountPath: /data
          name: data-volume
  volumes:
    - name: data-volume
      awsElasticBlockStore:
        volumeID: <AWS-VolumeID>
        fsType: ext4
```

## PV & PVC
* This involves the process of creaing a Persistent Volume which can then be assigned to a Persistent Volume Claim which will be attached to a node.
* The configuration & requirement of the PVC must match with the PV to be allocated. Insead of `hostPath`, we can also use `awsElasticBlockStore` too.
```yaml
apiVersion: v1
kind: PersistantVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadOnlyMany
  capacity:
    storage: 10Gi
  hostPath:
    path: /path
  persistantVolumeReclaimPolicy: Retain
```
* Create the PV by `kubcetl create -f pv.yaml`. 
Get exisitng PV by `kubectl get pv`.
* Create the PVC based on requirement
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
    # if required, match with a PV
  selector:
    matchLabels:
      comp: pv
      name: log-pv
```
* Add the config to the pod so that ut uses it
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - image: kodekloud/event-simulator
      name: event-simulator
      volumeMounts:
        - mountPath: /log
          name: log-volume
  # Change is here where we use persistentVolumeClaim instead of hostPath & give the name of the claim.
  volumes:
    - name: log-volume
      persistentVolumeClaim:
        claimName: claim-log-1
```

## Storage Class
* Here instead of using PV definition & creating a PV everytime, we will be defining a SC which will be dynamically providing storage based on PVC.
* Storage Class Definition contains the information about the provider & the provider specific configuration
```yaml
apiVersion: storage/k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
volumeBindingMode: WaitForFirstConsumer | Immediate
parameters: # Differs from provider to another
  type: pd-standard
  replication-type: none
```
* Use the SC in the PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: scName
```
* Use the PVC in Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logger-pod
spec:
  containers:
    - name: logger-container
      image: logger
      volumeMounts:
        - mountPath: /opt/data
          name: logger-volume
  volumes:
    - name: logger-volume
      persistentVolumeClaim:
        claimName: pvcName
```
