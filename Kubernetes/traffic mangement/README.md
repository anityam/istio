# Traffic Management
The traffic management concept of istio is an important concept in Istio. The traffic management rules helps to manage the connection flow between services in the kubernetes cluster. Istio uses envoy proxy sidecars to control the flow of traffic in the cluster.**Istio intercepts the traffic through envoy proxy and reroutes the traffic based on rules**

## Concept
Istio keeps a database of it's managed service and service endpoints in a registry called service registry. The envoy proxy uses this service registry to route the traffic. Istio has a mechanism of service discovery (Pilot) or [ServiceEntry](https://istio.io/latest/docs/reference/config/networking/service-entry/) configuration can be added to add the service to the service Registry.
Envoy proxy distributes traffic based on least load model. If there are more than two loadbalancer or host receiving the traffic than envoy routes the traffic to the host that is handling the least amount of traffic
Outside of the routes istio can also split traffic across instances 

## Apis
1. Virtual Service
2. Destination Rule
3. Service Entry
4. Gateways
5. Sidecars


### Virtual Service
This routes the traffic from to specific services at the backend. It can be used to define routes to various real services based on the rules. https://istio.io/latest/docs/concepts/traffic-management/#virtual-services

### Destination Rule 
This handles traffic to the specific destination. This is behind the virtual service once the traffic is directed to specific host then the destination service can apply the rules to the traffic

### Gateway
Gateway are how traffic comes to the cluster and then it can be routed to a virtual service

### Service Entry
This is usually used to define a traffic leaving a cluster where a specific call in the cluster can be routed to an entry point pointing to the external cluster.

### sidecars
To define a special rules for the sidecars used by the istio


## Exercise
### Initial setup
1. Create the cluster through kind using `kind create cluster --config kind/Config.yaml`
2. Create the istio deployment using istioctl
    1. Initialize istio operator `istioctl operator init`
    2. Install the istiooperator setting `k apply -f Kubernete/istio-setup/istiooperator.yaml`


### Bookinfo Example
In this we will setup a simple book example used in the istio.io 
1. Create a separate namespace where istio  injection is enabled `k apply -f Kubernetes/produc-test/namepsace.yaml`
2. Create a deployment for bookinfo `k apply -f kubernetes/produc-test/bookinfo.yaml`
3. Create a gateway to route the incoming traffic at the ingress to the productpage deployment `k apply -f Kubernetes/produc-test/gateway.yaml`
4. Once all deployment are complete you could hit the https://127.0.0.1:3000/productpage and see the website

In this deployment we create a microservice based deployment where each pods are handling a specific routes. The productpage handles the productpage and we are directing all traffic coming through ingress gateway by an option in 
```yaml 
# Kubernetes/produc-test/gateway.yaml
hosts:
    - "*"
  gateways:
    - bookinfo-gateway
  http:
    - match:
        - uri:
            exact: /productpage
        - uri:
            prefix: /static
        - uri:
            exact: /login
        - uri:
            exact: /logout
        - uri:
            prefix: /api/v1/products
      route:
        - destination:
            host: productpage
            port:
              number: 9080
```
so when any incoming traffic comes to bookinfo-gateway with the following uri then it is routed to productpage port 9080. The logs of incoming request can be seen at productpage


### Host calls changes
In this we will change the virtual service call to specific host i.e. `bookinfo.com`. For the specific call some adjustment has to be made in the local environment. Instead of doing a localhost call we need to do a dns based call. 
1. Edit the `/etc/hosts` to add lookup for `bookinfo.com` and `test1.com` to 127.0.0.1
2. Edit the virtualservice for bookinfo-gateway to  only route to /productpage for `bookinfo.com` 
```yaml
hosts:
    - "bookinfo.com"
  gateways:
    - bookinfo-gateway
  http:
    - match:
        - uri:
            exact: /productpage
        - uri:
            prefix: /static
        - uri:
            exact: /login
        - uri:
            exact: /logout
        - uri:
            prefix: /api/v1/products
      route:
        - destination:
            host: productpage
            port:
              number: 9080
```
With this the call never reaches to the productpage if you hit the `test1.com:3000/productpage` and will reach the pods if you do `bookinfo.com:3000/productpage`

Let's revert the above change and see 

```yaml
hosts:
    - "*"
  gateways:
    - bookinfo-gateway
  http:
    - match:
        - uri:
            exact: /productpage
        - uri:
            prefix: /static
        - uri:
            exact: /login
        - uri:
            exact: /logout
        - uri:
            prefix: /api/v1/products
      route:
        - destination:
            host: productpage
            port:
              number: 9080
```
With this change both will `test1.com` and `bookinfo.com` will hit the pods


### Make all calls go to v1
Looking at the setup in `kubernetes/produc-test/bookinfo.yaml` there are only reviews that has 3 version. A closer inspection can be done by installing a dashboard which can be done through  installing the `kiali` dashboard which is in the addons section of samples so

```bash
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system
```

Then 
```bash
istioctl dashboard kiali
```
and send some traffic to the cluster to capture the traffic by the dashboard 
```bash
for i in $(seq 1 100); do curl -s -o /dev/null "http://bookinfo.com:3000/productpage"; done
```
## Document
https://istio.io/latest/docs/concepts/traffic-management/ --> Traffic management
https://istio.io/latest/docs/reference/config/networking/service-entry/ --> Service Entry
https://www.istioworkshop.io/03-servicemesh-overview/introduction-service-mesh/


