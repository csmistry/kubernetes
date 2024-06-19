# Istio
- istio is a service mesh
- Service Mesh manages communication between microservices

Problem: Transitioning from a monolith to a microservice architecture
- suppose we are trying to deploy an online store using k8s cluster
- this online store will have microservices such as payments, db, inventory
- Each microservice will be deployed as a separate pod with the following config
    - each microservice will have its own business logic
    - microservices's need to be able to talk to each other
      - when we add a new microservice that microservices endpoint needs to be added to all other pods as well
    - we need some security logic for communication within the cluster
    - we also need monitoring for our services

- From a security standpoint once you in the cluster there is no protection because any service inside the cluster can talk to any other service inside the cluster
- you can setup firewalls and proxys outside the cluster to make it more secure but once you are in the cluster an attacker can access anything

- If you look at the list of things required for a mircoservice all the configuration logic will have to be impolemented for each microservice. This means that teams are actually spending more time configuring things rather than developing the product


Solution: 

Now instead of application developers doing the configuration we can move all that logic into a Sidecar Proxy

Sidecar Proxy
- This is a container that the k8s control plane will automatically inject into each microservice Pod
- The proxy container is resposible for:
  - handling networking logic
  - to communicate with other pods, the proxys will talk to each other

  So the network layer which includes the control plane and the proxys form the service mesh

  Feature: Traffic split (Canary Deployments)
  - When you release a new version of an application you dont want to instantly send all traffic
  - with canary deployments you can direct certain percentage of your traffic to the new application and monitor to make sure that it is working as expected


## Istio Architecture

- Remember, Service Mesh is just a pattern. Istio is an implementation of that pattern

istio has two components
1. Control Plane Component
- This is called istiod and its responsible for configuration, discovery, certificates etc. This is the component that injects the Envoy proxy into microserivce pods

2. Data Plane Component
- The istio proxy that gets injected into microservice pods is an Envoy Proxy. This is an open source project. They Envoy Proxies form the Data plane. 
- "Mesh traffic" is the communication that occurs between envoy proxies

- istio config is separate from deployment or serivce yamls
- istio uses CRDs to create k8s istio resources so that you can still use yaml to configure these resources as native k8s objects

Two main CRDs for configuring service-to-service communication

#### Virtual Service
- This defines how to route your taffic TO a specific service

#### Destination Rule
- Once you define your VirtualService you can define a DestinationRule which lets you configure what happens to the traffic for that destination service. So things like what kind of load balancing do you want for the traffic before it gets to the pods in your destination service

How do these configs get applied?

Once you create these CRs istiod will convert the highlevel routing rules into Envoy-specific configurations which are then propagated to the envoy proxy sidecars


Istio Features
- Configuration of proxies (through CRDs)
- Internal Registry for Services and their endpoints
  - This is dynamic so everytime a new microservice comes up istio will automatically register that endpoint
  - The Envoy proxies use this registry to query the endpoints so that they can send traffic to the relevant endpoints
- Istio also does certificate manaagement. Istio will generate certificates for all microservices in the cluster to allow secure TLS communication between all proxies in the cluster
- you can gather telemetry data from the proxies

Istio Ingress Gateway
- This is the entry point to the cluster. (alternative to an nginx ingress controller) 
- Cluster request comes in and it will first go through the Ingress Gateway
- Runs a pod in your cluster and acts as a load balancer. Gateway will direct traffic to your microservice by using the VirtualService defined
- You can configure istio gateway using the gateway CRD

Traffic Flow
1. Request comes to the cluster and first hits the istio ingress gateway
2. Ingress gateway evaluates the request and figures out where to send the traffic based on VirtualSerivces
3. Request is sent to the Envoy Proxy for at the VirtualService. 
4. Envoy Proxy then sends it to the actual service

The proxies will gather all the details about the traffic flow and send it to the istiod control plane so that we can have telemetry