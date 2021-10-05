# k8s-banking-testbed-argocd

## Running Bootstrapping ArgoCD & Istio

To bootstrap the app on Argo CD run the following command (add relative path as neccesary). 

'kubectl apply -k bootstrap-argocd/argocd-istio-bootstrap'

NOTE: ensure argocd is installed on your system and a cluster is setup before running the above command