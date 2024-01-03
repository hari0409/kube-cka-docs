# Creation of Ingress Service

## Creation of the Ingress Controller
* Create a namespace for the controller using the command `kubectl create namespace ingress-nginx`.
* Create a ConfigMap in the same namespace using the command `kubectl create cm ingress-nginx-controller -n ingress-nginx`
* Create two SA:
    * ingress-nginx `kubectl create sa ingress-nginx -n ingress-nginx`
    * ingress-nginx-admission `kubectl create sa ingress-nginx-admission -n ingress-nginx`
* Apply the two Roles `ingress-nginx` & `ingress-nginx-admission`.
* Apply the two ClusterRole `ingress-nginx` & `ingress-nginx-admission`.
* Apply the Rolebinding & the ClusterRoleBinding for the role & clusterrole `ingress-nginx` & `ingress-nginx-admission`.
* Deploy the file `ingress-controller.yaml`.

## Creation of the Ingress Resources
* In the required namespace where the required services exists, create the Ingress Resources.
* They can be done in two ways
    * Declarative Way `kubectl create ingress NAME --rule=host/path=service:port`
    * Imperative Way using the [file](/Complete-Ingress-Files/ingress-resource.yaml).

