# Istio

## About
Istio is a service mesh that will intercept the traffic between the pods and help to communicate the micro services


## Setup
Using the kind cluster for the local setup

### Structure
Kind --> [local k8 cluster ](./kind/)
samples --> example codes in [istio github repo](./samples/)
kubernetes --> our kubernetes code 

### Installation
#### Cluster
The cluster can be installed through `kind create cluster --config kind/Config.yaml`. Once the command is successful then query the cluster through `kind get clusters` and should be able to see a cluster called kind. The [config file](./kind/Config.yaml) defines two worker and one control plane with port mapping done for the control plane to access the apps in the cluster.

#### Istion Installation 
Istio can be setup using the istioctl tool which can be install through `brew install istioctl`. The istio installation can de done through the helm charts or using a yaml istiooperator template.

Through istioctl 
    1. Run the installation through istioctl by doing `istioctl install --set profile=demo -y`. This will install the istio components and a demo profile .More on profile at https://istio.io/latest/docs/setup/additional-setup/config-profiles/. Profile is basically the resources that will be installed to facilitate the service mesh.

Using istiooperator 
    1. Initialize the istio operator first ` istioctl operator init`
    2. apply the istooperator yaml file  `kubectl apply -f Kubernetes/Istio-setup/istiooperator.yaml`

## Documentation
https://istio.io/ --> Istio doc
https://github.com/istio/istio --> github
https://www.danielstechblog.io/running-istio-on-kind-kubernetes-in-docker/ --> istio in kind
https://tetrate.io/blog/what-is-istio-operator/ information on istio operator installation
https://istio.io/latest/docs/setup/additional-setup/config-profiles/ --> istio profile