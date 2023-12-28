# Monitoring & Logging in Kubernetes

## Monitoring Solution:
* This can be done by means of 3rd part implementation of `metrics-server` which workds by getting the resource utilisation from `cAdvisor` which runs on each node to get the data & sends to the kubeapi.
* This can be run in two methods:
    * If using minikube, `minikube addons enable metrics-server`.
    * Other Environment:
        * Download the code from git & deploy the objects
        ```bash
        # Download the YAML configurations
        git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
        # Deploy the resources
        kubectl create -f .
        ```
* Get the node resources using the command `kubectl top nodes` & for the pods using the command `kubectl top pods`.

## Logs from Kubernetes
* To get the logs from the containers in the pod, we can use the command `kubectl logs -f podName`
* When multiple containers are being used, we can use the command `kubectl logs -f podName containerName`
* To get the logs of dead container `kubectl logs podName -c containerName` or like above command too.
