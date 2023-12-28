# Application Lifecycle Management:

## Rollout & Rollback:
* To make changes to a deployment, make the changes in the deployment-config fiile & the use the command `kubectl apply -f deplo-config.yaml` or we can also change the image directly using the cmd `kubectl set image deploymentName containerName=imageName`
* To get the STATUS of rollout `kubectl rollout status deployment deploymentName`
* To get the HISTORY of rollout `kubectl rollout history deployment deploymentName`
* To UNDO rollout `kubectl rollout undo deployment deploymentName`
* The rolling update can be configured as
```yaml
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25% # Max number of addn nodes that can be created at a time
      maxUnavailable: 25% # Max number nodes that can be destroyed at a time
```

## Commands & Arguments:
* To override the CMD of an image, use `args: ["value"]` in the pod file.
* To overrise the ENTRYPOINT of an image, use `command: ["sleep"]` in the pod.

## Environemtnal Variable:
* This can be used in a container & can be specified in the pod in three ways:
  * Directly as KEY-VALUE pairs in the ENV field
  * By Using a ConfigMap & gettin the data from it by:
    * Mapping a configMap for the pod
    * Getting single value from the configMap alone.
* Directly as key-value pairs:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  labels:
    app: frontend
spec:
  containers:
    - name: frontend-container
      image: frontend-image
      env: 
        - name: DATABASE_URL
          value: mongodb://localhost:7766
        - name: ENV_NAME
          value: ENV_VALE
```
* ConfigMap can be created in three ways:
  * Using the --from-literal command `kubectl create configmap configmapName --from-literal=KEY1=VALUE1 -—from-literal=KEY2=VALUE2`
  * Using the --from-file command `kubectl create configmap configmapName --from-file=PATH_TO_FILE` where the file content will be:
  ```yaml
  KEY1: VALUE1
  KEY2: VALUE2
  ```
  * Using the ConfigMap Object File
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-configmap
  data:
    KEY1: VALUE1
    KEY2: VALUE2
  ```
* Getting the value as ConfigMap File using `envFrom` & `configMapRef`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  labels:
    app: frontend
spec:
  containers:
    - name: frontend-container
      image: frontend
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config # name given for the configmap in the metadata section.
```
* Getting a single value from ConfigMap:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  labels:
    app: frontend
spec:
  containers:
    - name: frontend-container
      image: frontend
      ports:
        - containerPort: 8080
      env:
        - name: ENV_NAME
          valueFrom:
            configMapKeyRef:
              name: app # name of configmap given in metadata
              key: KEY_IN_FILE
```

## Secrets in Kubernetes:
* Secrets are very similar to ENV. This can be provided in two ways:
  * Using --from-literal `kubectl create secret generic secretName --from-literal=key1=value1 --from-literal=key2=value2`
  * Using --from-file `kubectl create secret generic secretName --from-file=fileName.properties` where the file will have contents as:
  ```bash
  DB_HOST: mysql
  DB_USER: root
  DB_PASS: 1233
  ```
  * Using Secret Object Config:
  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: app-secret
  data:
    KEY1: VALUE1_ENCODED
    KEY2: VALUE2_ENCODED
  ```
  where we can encode the data using command `echo -n "value" | base64` & decode the data by `echo -n "valu" | base64 --decode`
* Injecting the secret into the pod
  * The whole file
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: frontend-pod
  spec:
    containers:
      - name: frontend-container
        image: frontend-image
        ports: 
          - containerPort: 8080
        envFrom:
          - secretRef:
              name: secret-name # given in metadata
  ```
  * Single value alone
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: frontend-pod
  spec:
    containers:
      - name: frontend-container
        image: frontend-image
        ports: 
          - containerPort: 8080
        envFrom:
          - secretRef:
              name: secret-name # given in metadata
  ```

## Multi Container Pods:
* This can be created by providing multiple containers as an array in the spec.
* We can have command executed in the containers of those pods using the command `kubectl exec podName containerName -it -- bash`

## InitContainers:
* This can be used for running the containers before the actuall application containers in the order specified that can be used for the configuration of the pods before the actual applicatin runs.
* Example Implementaiton would be:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: cName
      image: iName
  initContainers:
    - name: cName
      image: iName
      command: ["sleep","1000"]
```

## Encryption in Kube
* Create a `EncryptionConfiguration` Object by providing the encryption, key & value in the order to be performed like
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - identity: {} # plain text, in other words NO encryption
      - aesgcm:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
            - name: key2
              secret: dGhpcyBpcyBwYXNzd29yZA==
      - aescbc:
          keys:
            - name: key1
              secret: c2VjcmV0IGlzIHNlY3VyZQ==
            - name: key2
              secret: dGhpcyBpcyBwYXNzd29yZA==
```
* Then we can modify the kube-apiserver config YAML file by inserting the following at the requierd places
```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # add this line
    volumeMounts:
    ...
    - name: enc                           # add this line
      mountPath: /etc/kubernetes/enc      # add this line
      readOnly: true                      # add this line
    ...
  volumes:
  ...
  - name: enc                             # add this line
    hostPath:                             # add this line
      path: /etc/kubernetes/enc           # add this line
      type: DirectoryOrCreate             # add this line
  ...

```
* Here we can generate the random key by `head -c 32 /dev/urandom | base64`

