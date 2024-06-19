# Istio

A Service Mesh manages communication between microservices, it is a pattern software design. Istio is an implementaion of that pattern, which is why we often hear that Istio is a service mesh.

### Problem: Transitioning from a monolith to a microservice architecture
- Suppose we are trying to deploy an online store using k8s cluster. This online store will have microservices such as payments, db, inventory.
- Each microservice will be deployed as a separate pod with the following config:
    - Its own business logic
    - Ability to talk to other microservices
      - New microservice endpoints should be discoverable by all other microservices
    - Security logic for communication within the cluster
    ##### Why do we need security inside the cluster?
    From a security standpoint once you are in the cluster there is no protection because any service inside the cluster can talk to any other service inside the cluster. You can setup firewalls and proxys outside the cluster to make it more secure but once you are in the cluster an attacker can access anything
    - Monitoring for our services

All of the above logic needs to configured for each microservice. Thus, developers can end up spending more time on configuration rather than building the actual product.


### Solution: Istio

Now instead of application developers doing the configuration we can move all that logic into a Istio Sidecar Proxy

#### Istio Sidecar Proxy
- This is a container that the k8s control plane will automatically inject into each microservice pod
- This istio proxy is called the Envoy proxy and it is developed as an open source project
- The Envoy proxy is resposible for handling the networking logic that will allow communication between services both inside and outside the cluster

## Istio Architecture

Istio has 3 main components: 

1. Control Plane Component
- This component is istiod and its responsible for configuration, discovery, certificates etc. Istiod injects the Envoy proxy into microserivce pods

2. Data Plane Component
- The Envoy proxies form the data plane. 
- "Mesh Traffic" is the communication that occurs between envoy proxies

The network layer is then the combination of istiod in the control plane and the envoy proxies, which together form the Service Mesh.

3. Istio Ingress Gateway
- This is the entry point to the cluster. (alternative to an nginx ingress controller) 
- Cluster request comes in and it will first go through the Ingress Gateway
- Runs a pod in your cluster and acts as a load balancer. Gateway will direct traffic to your microservice by using the VirtualService defined
- You can configure istio gateway using the gateway CRD

### How does traffic flow?
1. Request comes to the cluster and first hits the istio ingress gateway
2. Ingress gateway evaluates the request and figures out where to send the traffic based on VirtualSerivces that are defined
3. Request is sent to the Envoy proxy for the VirtualService. 
4. Envoy proxy then sends it to the actual service

### How is Istio configured?
Istio config is separate from deployment or serivce yamls. Istio uses CRDs to create k8s istio resources that can be managed as native k8s objects

There are two main CRDs for configuring service-to-service communication:

#### 1. Virtual Service
This defines how to route your taffic TO a specific service

#### 2. Destination Rule
Once you define your VirtualService you can define a DestinationRule which lets you configure what happens to the traffic for that destination service. The DestinationRules allows you to configure things like what kind of load balancing do you want for the traffic before it gets to the pods in your destination service

#### How do these configs get applied?
Once you create these CRs istiod will convert the highlevel routing rules into Envoy-specific configurations which are then propagated to the envoy proxy sidecars

### Istio Features
1. Configuration of proxies (through CRDs)
2. Internal Registry for services and their endpoints
  - This is dynamic so everytime a new microservice comes up istio will automatically register that endpoint
  - The Envoy proxies use this registry to query the endpoints so that they can send traffic to the relevant endpoints
3. Certificate Management
  - Istio will generate certificates for all microservices in the cluster to allow secure TLS communication between all proxies in the cluster
4. Telemetry data from proxies
  - The proxies will gather all the details about the traffic flow and send it to the istiod control plane
- Traffic split (Canary Deployments)
  - When you release a new version of an app you dont want to instantly send all traffic to that new app
  - With canary deployments you can direct certain percentage of your traffic to the new app and monitor to make sure that it is working as expected

# Sources
[1] [Istio & Service Mesh - simply explained in 15 mins](https://www.youtube.com/watch?v=16fgzklcF7Y&list=PLEP50MzAcEUg5T8WdKvz3t4n5Zu5q5oDH&ab_channel=TechWorldwithNana)