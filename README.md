# k8s-banking-testbed-argocd

GitOps Repo for the services found in the k8s-banking-testbed repository

## Running Bootstrapping ArgoCD & Istio

1. Create A Cluster with traefik disabled use the config.yaml like below:

> k3d cluster create --config ./bootstrap-argocd/cluster/config.yaml

**NOTE:** a normal K3D cluster will not work as Istio Ingress LoadBalancer will never get assigned an IP Address.

2. To bootstrap the app on Argo CD run the following command (add relative path as neccesary).

> kubectl apply -k bootstrap-argocd/argocd-bootstrap/argocd-istio-bootstrap

**NOTE:** ensure argocd is installed on your system and a cluster is setup before running the above command

*You may need to rerun the above command as the Application CRD sometimes throws the following error.*

**'no matches for kind "Application" in version "argoproj.io/v1alpha1"'**

3. Run the following commands to get Argo Rollouts setup properly

> kubectl delete deployment argo-rollouts -n argocd
> kubectl create namespace argo-rollouts
> kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

4. Apply the 3 .yaml files to kubectl located here - https://github.com/Bilaltariq98/k8s-banking-testbed/tree/main/istio

> kubectl apply -k k8s-banking-testbed/istio/kiali.yaml
> kubectl apply -k k8s-banking-testbed/istio/grafana.yaml
> kubectl apply -k k8s-banking-testbed/istio/prometheus.yaml

You may need to rerun the above command as the CRD sometimes throws an error.

5. Use the following command to check the status of the pods in the Argo CD namespace. Ensure all of them are in the status 'Running'

> watch kubectl get pods -n argocd

6. Get the ARGO CD Admin password with the following command.

> kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

6. To access the testbed application:

a.) Get the Istio Ingress IP via the following

> kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

b.) Update /etc/hosts with the following entries (replace ISTIO_INGRESS_IP with your IP from the above output)

> ISTIO_INGRESS_IP banking-testbed.com banking-testbed-preview.com argo.cd

Access the testbed application by accessing http://banking-testbed.com

Access the preview (green) instance of the application during code promotion via http://banking-testbed-preview.com

Access argo cd by accessing http://argo.cd

# Kafka Setup

Setup a Kafka operator namespace

> kubectl create ns kafka-operator

Add Strimzi Helm Repository and update local index

> helm repo add strimzi https://strimzi.io/charts/
> helm repo update

Install Strimzi to kafka-operator namespace

> helm install strimzi strimzi/strimzi-kafka-operator --namespace kafka-operator

Sync the kafka applications inside the Argo CD UI (cluster / topics)

## Finding IP:Port for accessing kafka topics

### Port

> kubectl get service kafka-cluster-kafka-external-bootstrap -n kafka-operator -o=jsonpath='{.spec.ports[0].nodePort}{"\n"}'

### IP

> kubectl get nodes --output=jsonpath='{range .items[*]}{.status.addresses[?(@.type=="InternalIP")].address}{"\n"}{end}'

If you download the Kafka binaries you will be able to run the following commamds in two different terminals to test PUB/SUB to the kafka topic kafka-topic

**Producer**

> bin/kafka-console-producer.sh --broker-list 172.20.0.6:30002 --topic kafka-topic

**Consumer**

> bin/kafka-console-consumer.sh --bootstrap-server 172.20.0.6:30002 --topic kafka-topic --from-beginning

TODO: Incorporate [Avro Schema](https://github.com/confluentinc/cp-helm-charts/tree/v5.4.0/charts/cp-schema-registry) or try using [this](https://github.com/lsst-sqre/strimzi-registry-operator)
TODO: Think about the way [kafka is exposed](https://strimzi.io/blog/2019/04/17/accessing-kafka-part-1/)
