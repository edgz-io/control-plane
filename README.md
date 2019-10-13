# Control Plane Setup
This is a basic set of steps for getting openfaas running with istio service mesh.

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
1. `helm init --service-account tiller`
1. `helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -`
1. Re-run until 23 is hit (all CRDs installed) `kubectl get crds | grep 'istio.io' | wc -l`
1. Verify running `kubectl get pods -n istio-system`

### Install OpenFaaS for Serverless Capabilities
Reference: https://github.com/openfaas/faas-netes/blob/master/chart/openfaas/README.md
1. `brew install faas-cli`
1. `kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml`
1. `helm repo add openfaas https://openfaas.github.io/faas-netes/`
1. `PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)`
1. `kubectl -n openfaas create secret generic basic-auth --from-literal=basic-auth-user=admin --from-literal=basic-auth-password="$PASSWORD`
1. `helm repo update && helm upgrade openfaas --install openfaas/openfaas --set ingress.enabled=true --set serviceType=LoadBalancer --namespace openfaas --set basic_auth=true --set functionNamespace=openfaas-fn`
1. `export OPENFAAS_URL=http://127.0.0.1:31112`
1. `echo -n $PASSWORD | faas-cli login -g $OPENFAAS_URL -u admin --password-stdin`
1. `faas-cli version`
