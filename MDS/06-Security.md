# Security in Kubernetes:

## Certificates in Kubernetes
* There are two types of certificates:
    * Server Certificates
    * Client Certificates
* Client Certificates:
    * This can be created for the various components & they can associated wit groups too in case of admin-user & nodes in Server Certificates.
    * This is done in three steps. It includes:
        * Generating the Key
        ```bash
        openssl genrsa -out component.key
        ```
        * Creating CSR
        ```bash 
        openssl req -new -key component.key -subj "/CN=component/" -out component.csr
        ```
        * To specify the group, we can specify with the subject by:
        ```bash 
        openssl req -new -key admin.key -subj “/CN=kube-admin/O=system:masters" -out admin.csr
        ``` 
        * Signing the CSR
        ```bash
        openssl x509 -req -in component.csr -CA ca.crt -CAkey ca.key -out component.crt
        ```
    * This can then be used in CURL command like `curl https://kube-apiserver:6443/api/v1/pods --key admin.key --cert admin.cert --cacert ca.cert` or used in a config file like:
    ```yaml
    apiVersion: v1
    clusters:
    - cluster: 
        certificate-authority: ca.crt
        server: https://kube-apiserevr:6443
        name: kubernetes
    kind: Config
    users:
    - name: kubernetes-admin
        users:  
        client-certificate: admin.crt
        client-key: admin.key
    ```
* Server Certificates:
    * This needs to be created for three components:
        * ETCD Cluster
        * kube-apiserver
        * kubelet server
    * To provide the CNF file for any alternative domain configuration as required for kube-apiserver, use the command `openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr --config openssl.cnf` where the openssl.cnf file would be like
    ```conf
    [req]
    req_extensions = v3_req
    distinguished_name = req_distinguished_name
    [ v3_req ]
    kasicContraints = CA:FALSE
    keyUsage = nonRepudiation
    subjectAltName = @alt_name
    [ alt_name ]
    DNS.1 = kubernetes
    DNS.2 = kubernetes.default
    DNS.3 = kubernetes.default.svc
    DNS.4 = kubernetes.default.svc.cluster.local
    IP.1 = 10.96.0.1
    IP.2 = 172.17.0.87
    ```
    * Fields required for ETCD are:
    ```bash 
    --key-file
    --cert-file
    --trusted-ca-file

    --peer-cert-file
    --per-client-cert-auth
    --peer-key-file
    --peer-trusted-ca-file
    ```
    * Fields required for kube-apiserver are:
        * Kube-apiserver — ETCD Server Client Certificates, Key & CA
        ```bash
        --etcd-cafile
        --etcd-keyfile
        --etcd-certfile
        ```
        * Kuber-apiserver — Kubelet Server Client Certificate, Key & CA
        ```bash
        --kubelet-certificate-authority
        --kubelet-client-certificate
        --kubelet-client-key
        ```
        * Kube-apiserever Server Certificate, Key & CA
        ```bash
        --client-ca-file
        --tls-cert-file
        --tls-private-key
        ```
* To view the certificate:
    * Open the Pod Config file
    * Check the field for the cert
    * View the cert using the command `openssl x509 -in /path-to-cert -text -noout`
* All the certificates are located in the folder `/etc/kubertnetes/pki`. In here based on the components, all the certificates are located. For example the cert for ETCD are present under `/etc/kubernetes/pki/etcd`.

## Certificate Management:
* This can be used for creating client authentication certificates for users. 
* Geneate the key
```bash
openssl genrsa -out user.key 2048
```
* Create a CSR
```bash
openssl req -new -key user.key -subj "/CN=user" -out user.csr
```
* Create a CSR Object file & apply it
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user
spec:
  group:
    - system:masters
    - system:authenticated
  request: BASE64_ENCODED_CSR
  usages:
    - server auth
    - client auth
  signerName: kubernetes.io/kube-apiserver-client
```
```bash
kubectl create -f csr.yaml
```
* Get the CSRs
```bash
kubectl get csr
```
* Accept or Deny the CSR
```bash
kubectl certificate approve|deny user
```
* Get the certificate signed, we can get the YAML file & the result will be in the field `certificate`
```bash
kubectl get csr csrName -o yaml
```
* Encoding & Decosing can be done by `cat file.csr | base64 -w 0` & decode by `echo BASE64 | base64 --decode`

## KubeConfig
* This is the configuration that is used to configure the kubernetes to access various clusters as various users based on context.
* This done with the help of config file in `/$HOME/.kube.config` & this have three section
    * Cluster
    * User
    * Context (Combination of user & cluster to determine which user can access which cluster.)
```yaml
apiVersion: v1
kind: Config

current-context: user@cluster

clusters:
  - name: cluster-name
    cluster: 
      certification-authority: ca.crt
      server: https://my-kubecluster:6443

context:
  - name: kube-admin@cluster-name
    context: 
      cluster: cluster-name
      user: kube-admin
      namespace: default_namespace_to_enter

users:
  - name: kube-admin
    user:
      client-certificate: admin.crt
      client-key: admin.key
```
* The `current-context` is used to define the default context & the default namespace can be defined for each context.
* These configuration can be accessed & modified by `kubectl config` command.
    * To view the current config `kubectl config view`.
    * To view a certain config `kubectl config --kubeconfig /configpath view`
    * To perform any action, we can do `kubectl config <command>` which will work on defualt context & `kubectl config --kubeconfig /path-config <command>` on the custom one.
    * To make the custom one the default one, replace the content of /root/.kube/config with the custom content.
    * Change context using command `kubcetl config use-context contextName`
    Create new context using the command `kubectl config set-context contextName`
* All these commands will directly update the config file.

## API Groups
* There are many API endpoints which can be used to acces the kubernetes cluster. This endpoints includes:
    * metrics
    * healthz
    * version
    * api
    * apis
    * logs
* `api` & `apis` are the main component responcible for cluster functionality.

## Authorisation:
* This is done by 4 ways:
    * Node Authorizer (Authorisation for the kubelet node systems:node for accessing the kube-apiserver)
    * ABAC (Assigned for each user based on requirement)
    * RBAC (Assigna role & permissions for each role)
    * Webhooks (3rd Party Authentication)
* RBAC Namespaced=True:
    * When we want to make config for namespace resources, we need to make `Role` & `RoleBdining`.
    * Role would require `apiGroups`, `resources`, `verbs`,`resourceNames` to which we need to give access.
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: role-name
    rules:
      - apiGroups: [""]
        resources: [“pods”,”pods/logs”]
        verbs: ["list","get","create","update","delete"]
      - apiGroups: [""]
        resources: ["configmaps"]
        verbs: ["create"]
      - apiGroups: [""]
        resources: ["rs"]
        verbs: ["*"]
    ```
    * Rolebinding would require the role & the users/groups as subjects.
    ```yaml
    apiVerion: rbac.authorization.k8s.io
    kind: RoleBinding
    metadata:
      name: userName-roleName-binding
      namespace: development
    subjects:
      - kind: User
        name: userName
        apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: developer
      apiGroup: rbac.authorization.k8s.io
    ```
* RBAC Namespaced=False:
    * Here we would be creating the same but for the cluster resources. We can also do for the namespace resource which will lead the resource under all the namespace to be accessible.
    * This is given as ClusterRole
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      creationTimestamp: "2023-12-28T07:44:25Z"
      name: storage-admin
      resourceVersion: "927"
      uid: 51a0ddd6-3c35-4860-a591-363337c2adce
    rules:
    - apiGroups:
      - ""
      resources:
      - persistentvolumes
      verbs:
      - '*'
    - apiGroups:
      - storage.k8s.io
      resources:
      - storageclasses
      verbs:
      - '*'
    ```
    * Then we will give the ClusterRoleBinding:
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: michelle-storage-admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: storage-admin
    subjects:
    - apiGroup: rbac.authorization.k8s.io
      kind: User
      name: michelle
    - apiGroup: rbac.authorization.k8s.io
      kind: Group
      name: system:masters
    ```
* We can create the Role, ClusterRole using the command `kubectl create -f role.yaml | cluster-role.yaml`.
* We can create the RoleBinding or ClusterRoleBinding using the command `kubectl create -f rb.yaml | crb.yaml`.
* To check if a user can do a particular action
```bash
kubectl can-i verb resource --as userName --namespace namespaceName
```
* To check the resources under namespace `kubectl api-resources namespaced=true` & for cluster resources, `kubectl api-resources namespaced=false`.

## Service Accounts
* This is used by application to access the kube cluster. 
* Steps in using a Service Account:
  * Create a Service Account using the command `kubectl create sa saName`. 
  * Create a Secret Token either by TokenRequest API or Secret Token Config file based on requirement.
    * For TokenRequest API, use the command `kubectl create token saName`.
    * For Service Account Token, 
    ```yaml
    apiVersion: v1
    kind: Secret
    type: kubernetes.io/service-account-token
    metadata: 
      name: serviceAccountToken
      annotations:
        kubernetes.io/service-account/name: saName
    ```
  * Map the SA with a Role & RoleBinding by making the `kind: ServiceAccount` in the rolebinding. 
  * Update the SA of the pod/deployment to be the created SA. This step ensures that the token is mounted to the pod/deployment. 
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:   
    name: pod-name
  spec:
    containers:
      - name: pod-container
        image: image-name
        ports:
          containerPort: 8080
    serviceAccountName: SA-name
    automountServiceAccountToken: true
  ```
  * Acces the token from the mounted folder.
