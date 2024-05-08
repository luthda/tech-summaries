# Minikube

## What is Minikube?

Minikube is a tool that allows you to run Kubernetes locally on your machine. It sets up a single-node Kubernetes cluster that is great for development and testing purposes.

## How to set up Minikube?

Pre-requisites: kubectl, Docker, kustomize

1. Install Minikube: Download and install Minikube by following the instructions on the official Minikube website.
2. Start Minikube: Use the minikube start command to start Minikube and create a local Kubernetes cluster.
3. Interact with Minikube: You can interact with your Minikube cluster using kubectl, the Kubernetes command-line tool. Use `kubectl get nodes` to see the nodes in your cluster.

##  Important Kubernetes concepts for this summary

###  Deplyoment

A Deployment in Kubernetes is a resource that manages the deployment of a Pod or a group of Pods. It ensures that a specified number of replicas of a Pod are running at any given time. The Deployment resource provides features like rolling updates, rollbacks, and scaling. It also handles the lifecycle of the Pods, such as creating, updating, and deleting them. The Deployment resource is responsible for maintaining the desired state of the application and ensuring that the actual state matches the desired state.

###  Service

A Service is an abstraction that defines a logical set of Pods and a policy by which to access them. It provides a stable endpoint for accessing the Pods and enables network communication between the Pods and other resources within the cluster.

A Service can be used to expose a single Pod or a group of Pods as a network service. It provides a stable IP address and DNS name that can be used to access the Pods. The Service acts as a load balancer, distributing incoming network traffic across the Pods.

Services can be of different types, such as ClusterIP, NodePort, and LoadBalancer. The ClusterIP type makes the Service accessible only within the cluster, while the NodePort type exposes the Service on each node's IP address at a static port. The LoadBalancer type provisions a cloud load balancer to route external traffic to the Service.

Services provide a level of abstraction and decoupling between the application and the underlying Pods, allowing for easier scaling, fault tolerance, and networking.

### Persistent Volume and Persistent Volume Claim

In Kubernetes, a Persistent Volume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster that represents storage that can be used by Pods.

A Persistent Volume Claim (PVC) is a request for storage by a user. It is similar to a Pod, in that it describes storage requirements and access modes. A PVC allows a user to request specific storage resources and the cluster satisfies the request with the help of the PV.

How it works:

1. An administrator creates one or more Persistent Volumes (PVs) in the cluster, which represent physical storage resources like network-attached storage (NAS), network-attached storage (NFS), or cloud-based storage.
2. Users create Persistent Volume Claims (PVCs) to request specific storage resources. The PVC specifies the desired storage size, access mode (e.g., ReadWriteOnce, ReadWriteMany), and other properties.
3. The Kubernetes scheduler matches the PVCs with available PVs based on the requested storage properties.
4. Once a PVC is bound to a PV, the PV is reserved for the PVC and cannot be used by other PVCs.
5. The PVC can then be used by Pods to persist data even if the Pods are rescheduled or restarted.

Persistent Volumes and Persistent Volume Claims provide a way to manage storage in Kubernetes in a scalable and flexible manner. They allow for dynamic provisioning of storage, data persistence across Pod restarts or failures, and easy migration of data to different storage systems.

###  Config Map

A ConfigMap is an object used to store configuration data in key-value pairs. It allows you to decouple configuration from container images, making it easier to manage and update application configurations.

A ConfigMap is a collection of configuration data that can be consumed by Pods or other Kubernetes resources. It is typically used to store configuration files, properties, or environment variables that need to be passed to applications running in Pods.

###  Ingress

In Kubernetes, an Ingress is an API object that manages external access to services within a cluster. It provides a way to route incoming HTTP and HTTPS traffic from the internet to the appropriate Kubernetes Service based on the hostname or URL path.

An Ingress acts as an entry point to the cluster, allowing external clients to access services running inside the cluster. It acts as a reverse proxy, forwarding incoming traffic to the appropriate backend service based on the rules defined in the Ingress resource.

An Ingress resource can have multiple rules, each specifying a hostname or URL path and the corresponding backend service to route traffic to. It can also provide load balancing and SSL/TLS termination capabilities.

Ingress controllers, such as Nginx or Traefik, watch for changes in the Ingress resources and update their configuration accordingly. They handle the routing of traffic to the appropriate backend services based on the defined rules.

### ClusterIP

A ClusterIP is a Kubernetes Service type that exposes the Service on a cluster-internal IP. This means that the Service is only reachable from within the cluster.

### Kustomize

Kustomization is a file used by the Kustomize tool to describe how to build, customize, and deploy Kubernetes YAML configurations. It allows you to define and manage multiple YAML configurations as a single unit.

Kustomization files are used to define how Kustomize should apply patches, overlays, and other customizations to a set of YAML configurations. They provide a way to manage complex deployments by breaking them down into smaller, reusable components.

## Setup Postgres and Adminer on local environment in Minikube

Move to src-folder and run the following commands:

### Deploy Postgres on Minikube

This command deployes all postgres resources in the kustomization.yaml:

```bash
kubectl apply -k postgres/
```

### Deploy Adminer on Minikube

This command deployes all adminer resources in the kustomization.yaml:

```bash
kubectl apply -k adminer/
```

### Deploy ingress on Minikube

This command deployes the ingress-controller:

```bash
kubectl apply -k ingress/ingress-controller.yaml
```

###  Modify host file

Add the following lines to the host file:

```bash
127.0.0.1 adminer.k8s.com
```

###  Open tunnel for ingress

Open a new terminal and run the following command:

```bash
minikube tunnel
```

Open adminer in browser and go to `http://adminer.k8s.com`.
Connect to postgres database by using credentials from configmap.yaml in postgres folder.
