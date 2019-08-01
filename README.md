# Metallb ON-PREMISE with Cilium and Nginx ingress controller 

Kubernetes implementation in the cloud services like Amazon (EKS), Google (GKE) or Azure (AKS) provides out of the box capabilities like Multi-Master High Availability, Ingress Load Balancer (to handle in the traffic from the internet), Network Storage, and launching worker nodes with different hardware requirements. 

All these facilities will NOT available if you install Kubernetes Clusters On-Premise if the infrastructure team uses an IaaS (Infrastructure as a Service) and builds the Kubnernetes cluster on bare metal. 

This section is will focus on how to deploy an Ingress enabled Load Balancer (at the Gateway) to handing the incoming traffic to the cluster. 

Bare metal cluster operators have left with two lesser tools to bring user traffic into their clusters, “NodePort” and “externalIPs” services. Both of these options have significant downsides for production use, which makes bare metal clusters second class citizens in the Kubernetes ecosystem. (From metallb web site).

MetalLB is a load-balancer implementation for bare metal Kubernetes clusters, using standard routing protocols.


## MetalLB requires the following to function:

1) A Kubernetes cluster, running Kubernetes 1.13.0 or later, that does not already have network load-balancing functionality.

2) A cluster network configuration that can coexist with MetalLB.

3)  IPv4 addresses for MetalLB .

4) Depending on the operating mode, you may need one or more routers capable of speaking BGP.


## 1. Kubernetes Setup

A Kubernetes cluster: v1.15.1 (3 node cluster) is already set ready.


## 2. Install Cilium Network Driver 

cluster network configuration : cilium 

kubectl create -f https://raw.githubusercontent.com/cilium/cilium/v1.5/examples/kubernetes/1.14/cilium.yaml



## 3. Install Metallb 

1) kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.1/manifests/metallb.yaml

(note: reference https://metallb.universe.tf/installation/)

This will deploy MetalLB to your cluster, under the metallb-system namespace. 

### 3.1 The components in the manifest are:

1) The metallb-system/controller deployment.( This is the cluster-wide controller that handles IP address assignments.)

2) The metallb-system/speaker daemonset. (This is the component that speaks the protocol(s) of your choice to make the services reachable)

3)Service accounts for the controller and speaker, along with the RBAC permissions that the components need to function.

![Screenshot from 2019-07-30 11-12-16](https://user-images.githubusercontent.com/30106168/62108245-35102f80-b2c7-11e9-996e-4542a9d6d607.png)

verify the speaker and controller are running state:

![Screenshot from 2019-07-30 11-12-40](https://user-images.githubusercontent.com/30106168/62108461-b962b280-b2c7-11e9-97fc-5ace03d32aef.png)

### 3.2 Add configMap

MetalLB’s components  will remain idle until you define and deploy a configmap.(for demo we will be using layer2 configuration)
 
kubectl apply -f  https://raw.githubusercontent.com/meta-magic/metallb-baremetal-example/master/metallb_install/configMap_example.yml

## 4. Install Nginx Ingress Controller

1) kubectl create -f https://raw.githubusercontent.com/meta-magic/metallb-baremetal-example/master/nginx-ingress/nginx_controller_install.yml


2)kubectl create https://raw.githubusercontent.com/meta-magic/metallb-baremetal-example/master/nginx-ingress/nginx_ingress_svc.yml

![Screenshot from 2019-07-30 11-20-44](https://user-images.githubusercontent.com/30106168/62110414-f6c93f00-b2cb-11e9-8cea-310aff24eb37.png)


## 5. Create demo of hello-world
1) create a namespace  helloworld

kubectl create -f https://raw.githubusercontent.com/meta-magic/metallb-baremetal-example/master/helloworld_example/hello-world-ns.yml

2) create a pod 

kubectl create -f https://raw.githubusercontent.com/meta-magic/metallb-baremetal-example/master/helloworld_example/hello-pod.yml

3) create a cluster ip svc 

kubectl create -f https://raw.githubusercontent.com/meta-magic/metallb-baremetal-example/master/helloworld_example/hello-svc.yml

![Screenshot from 2019-07-30 11-29-11](https://user-images.githubusercontent.com/30106168/62110568-57f11280-b2cc-11e9-961a-e3d51014d268.png)

4) create a  ingress 
kubectl create -f https://raw.githubusercontent.com/meta-magic/metallb-baremetal-example/master/helloworld_example/hello-ing.yml

## 6. Verify the ip of ingress
kubectl get ing -n helloworld

![Screenshot from 2019-07-30 11-29-45](https://user-images.githubusercontent.com/30106168/62110702-a2728f00-b2cc-11e9-8298-1d75ced33da4.png)

## 7. Access url http://192.168.2.8 (ip of ingress)



![Screenshot from 2019-07-30 13-33-04](https://user-images.githubusercontent.com/30106168/62114793-f97c6200-b2d4-11e9-9e5a-23d00eab4790.png)

