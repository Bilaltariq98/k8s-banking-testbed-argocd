# k8s-banking-testbed-argocd
GitOps Repo for the services found in the k8s-banking-testbed repository 

## Running Bootstrapping ArgoCD & Istio

1. Create A Cluster with traefik disabled use the config.yaml like below:

>k3d cluster create --config ./bootstrap-argocd/cluster/config.yaml 

**NOTE:** a normal K3D cluster will not work as Istio Ingress LoadBalancer will never get assigned an IP Address. 

2. To bootstrap the app on Argo CD run the following command (add relative path as neccesary). 

>kubectl apply -k bootstrap-argocd/argocd-bootstrap/argocd-istio-bootstrap

**NOTE:** ensure argocd is installed on your system and a cluster is setup before running the above command

*You may need to rerun the above command as the Application CRD sometimes throws the following error.*

**'no matches for kind "Application" in version "argoproj.io/v1alpha1"'**

3. Run the following commands to get Argo Rollouts setup properly 

>kubectl delete deployment argo-rollouts -n argocd

>kubectl create namespace argo-rollouts

>kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

4. Apply the 3 .yaml files to kubectl located here - https://github.com/Bilaltariq98/k8s-banking-testbed/tree/main/istio

>kubectl apply -k k8s-banking-testbed/istio/kiali.yaml

>kubectl apply -k k8s-banking-testbed/istio/grafana.yaml

>kubectl apply -k k8s-banking-testbed/istio/prometheus.yaml


You may need to rerun the above command as the CRD sometimes throws an error.


5. Use the following command to check the status of the pods in the Argo CD namespace. Ensure all of them are in the status 'Running'

>watch kubectl get pods -n argocd

6. Get the ARGO CD Admin password with the following command.

>kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


5. port forward Argo CD and access it from the port forwarded URL (localhost:8081 in my case) 

>kubectl port-forward svc/argocd-server -n argocd 8081:443
