# k8s-banking-testbed-argocd

## Running Bootstrapping ArgoCD & Istio

1. Create A Cluster with traefik disabled use the config.yaml like below:

'k3d cluster create --config ./bootstrap-argocd/cluster/config.yaml' 

NOTE: a normal K3D cluster will not work as Istio Ingress LoadBalancer will never get assigned an IP Address. 

2. To bootstrap the app on Argo CD run the following command (add relative path as neccesary). 

'kubectl apply -k bootstrap-argocd/argocd-istio-bootstrap'

NOTE: ensure argocd is installed on your system and a cluster is setup before running the above command
