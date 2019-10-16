# Control Plane Setup
After running the below steps you will have:
1. Local kubernetes running
1. Istio CP w/ service mesh
1. UIs: Kiali (mesh visualizer), Jaeger (trace exploration & comparisons), Grafana (metrics)
1. OpenFaaS installed with one namespace for platform and second for functions, each function in service mesh (also OpenFaaS UI)

#### TODO
- [ ] Fully script local
- [ ] Full helm chart for all in one production
- [ ] Move in service for schemaless

## Local Setup

### Install minikube & setup hypervisor
1. `brew install kubectl`
1. `brew install kubernetes-helm`
1. `brew cask install minikube`
1. `brew install hyperkit`
1. `minikube config set vm-driver hyperkit`
1. `minikube config set memory 8000`
1. `minikube config set cpus 4`
1. Startup minikube - `minikube start`
1. Launch kubernetes dashboard `minikube dashboard`


### Install Istio w/ Tiller
1. `curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.3.2 sh -`
1. `cd istio-1.3.2`
1. `export PATH=$PWD/bin:$PATH`
1. `kubectl create namespace istio-system`
1. `kubectl --namespace kube-system create sa tiller`
1. `kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller`
1. `helm init --service-account tiller --override spec.selector.matchLabels.'name'='tiller',spec.selector.matchLabels.'app'='helm' --output yaml | sed 's@apiVersion: extensions/v1beta1@apiVersion: apps/v1@' | kubectl apply -f -`
1. `helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -`
1. Re-run until 23 is hit (all CRDs installed) `kubectl get crds | grep 'istio.io' | wc -l`
1. Verify running `kubectl get pods -n istio-system`
1. Enable automatic sidecar injection - `kubectl label namespace default istio-injection=enabled --overwrite`
1. `helm template install/kubernetes/helm/istio --name istio --namespace istio-system --values install/kubernetes/helm/istio/values-istio-demo.yaml | kubectl apply -f -`

Check that all containers running are "ready" before moving on.  There will be 3 in `completed` state which is expected. 
`ubectl get pods -n istio-system`
    

### Install OpenFaaS for Serverless Capabilities
Reference: https://github.com/openfaas/faas-netes/blob/master/chart/openfaas/README.md
1. `brew install faas-cli`
1. `kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml`
1. `helm repo add openfaas https://openfaas.github.io/faas-netes/`
1. `PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)`
1. `kubectl -n openfaas create secret generic basic-auth --from-literal=basic-auth-user=admin --from-literal=basic-auth-password="$PASSWORD"`
1. `helm repo update && helm upgrade openfaas --install openfaas/openfaas --set ingress.enabled=true --set serviceType=LoadBalancer --namespace openfaas --set basic_auth=true --set functionNamespace=openfaas-fn`
1. `export OPENFAAS_URL=http://127.0.0.1:31112`
1. `echo -n $PASSWORD | faas-cli login -g $OPENFAAS_URL -u admin --password-stdin`
1. `faas-cli version`


### Install MySQL Cluster
1. `helm repo add presslabs https://presslabs.github.io/charts`
1. `helm install presslabs/mysql-operator --name mysql-operator`


### Load Backplane Service
https://github.com/edgz-io/edgz-backend (in progress)


### Baseline Functions (core functions to be installed in OpenFaaS)
...


## Explore Service Mesh
`istioctl dashboard kiali`

## Explore Metrics
`istioctl dashboard grafana`

## Explore Traces
`istioctl dashboard jaeger`

## Add a test function to openfaas
`http://127.0.0.1:31112/ui/`

## Add MySQL Cluster
```
Cluster.yaml

apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque
data:
  ROOT_PASSWORD: bm90LXNvLXNlY3VyZQ==
  # USER: <your app user base64 encoded>
  # PASSWORD: <your app password base64 encoded>
  # DATABASE: <your app database base64 encoded>

apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlCluster
metadata:
  name: test
spec:
  replicas: 1
  secretName: test-secret
 ```
 
 ```
 $ kubectl apply -f cluster.yaml
secret/test-secret created
mysqlcluster.mysql.presslabs.org/test created
```
