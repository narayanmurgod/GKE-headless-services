# GCP Google Kubernetes Engine (GKE) Headless Service

## Overview

This repository demonstrates how to implement a **Headless Service** in **Google Kubernetes Engine (GKE)**, including steps to set up and deploy a Kubernetes cluster with both **ClusterIP** and **Headless Service** configurations.

### Key Concepts:
- **ClusterIP**: A type of Kubernetes Service that provides a stable internal IP address for accessing services within a Kubernetes cluster.
- **Headless Service**: A service without a cluster IP. It routes traffic directly to individual pods, rather than a load balancer, useful for stateful applications like databases.

## Steps

### Step 00: Pre-requisites

1. **Verify if GKE Cluster is created**
   - Check if your Google Kubernetes Engine cluster is already created.
   
2. **Verify if kubeconfig for kubectl is configured**
   - Ensure that `kubectl` is configured to interact with your GKE cluster by running the following command:

     ```bash
     gcloud container clusters get-credentials <CLUSTER-NAME> --region <REGION> --project <PROJECT>
     ```

     Replace `<CLUSTER-NAME>`, `<REGION>`, and `<PROJECT>` with your GKE cluster details.


3. **List GKE Kubernetes Worker Nodes**:
   Verify that your nodes are running correctly.

   ```bash
   kubectl get nodes
   ```

# Kubernetes ClusterIP and Headless Service

## Step 01: Introduction

### Why Headless Service?
Headless Services are particularly useful for stateful applications where you need pods to communicate with each other directly. In these cases, using a ClusterIP service with a load balancer would not be ideal, as you want clients or other pods to interact directly with individual pod IPs rather than through a load balancer.

**Common Use Cases for Headless Service**:
- **StatefulSets**: Pods in StatefulSets require stable DNS names for reliable identification, and headless services allow this direct communication.
- **Databases**: For databases like Cassandra, MongoDB, and others, a headless service is often required for direct pod communication, ensuring there is no load balancing that could disrupt consistency.

---

## Step 02: Define Kubernetes Deployment (`deployment.yaml`)

In this step, we will define a **Kubernetes Deployment** for a simple application. This deployment will create 4 replicas of the app, ensuring scalability and fault tolerance.

### Deployment YAML Configuration

This YAML file will create a deployment with 4 replicas, each running a container of a sample application. The application is exposed through port `8080` inside each container.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      name: my-app-pod
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app-container
          image: us-docker.pkg.dev/google-samples/containers/gke/hello-app:2.0
          ports:
            - containerPort: 8080
```
# Step-03: Kubernetes ClusterIP Service Configuration

This file defines a Kubernetes `ClusterIP` service for an application called `my-app`. The service exposes the application to other pods within the Kubernetes cluster.

### Service YAML Configuration

```yaml
apiVersion: v1
kind: Service 
metadata:
  name: my-app-clusterip-service
spec:
  type: ClusterIP # ClusterIP, # NodePort, # LoadBalancer, # ExternalName
  selector:
    app: my-app
  ports: 
    - name: http
      port: 80 # Service Port
      targetPort: 8080 # Container Port
```
# Step-04: Kubernetes Headless Service Configuration

This file defines a Kubernetes **Headless Service**, which does not assign a cluster IP. The service directly routes traffic to pods using their IPs.

### VERY IMPORTANT NOTE
When using a Headless Service, we should use both the **Service Port** and **Target Port** as the same.

A Headless Service directly sends traffic to a Pod with the Pod IP and Container Port. DNS resolution directly happens from the Headless Service to the Pod IP.

### Headless Service YAML Configuration

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-headless-service
spec:
  #type: ClusterIP # ClusterIP, # NodePort, # LoadBalancer, # ExternalName
  clusterIP: None
  selector:
    app: my-app
  ports:
    - name: http
      port: 8080 # Service Port
      targetPort: 8080 # Container Port
```
# Step-05: Deploy Kubernetes Manifests

This step involves deploying the Kubernetes manifests, checking the status of deployments, pods, and services, and observing the behavior of the Headless Service.

## Deploy Kubernetes Manifests

To deploy the Kubernetes manifests, use the following command:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f clusterip-service.yaml
kubectl apply -f headless-service.yaml 
kubectl apply -f curl-pod.yaml
```
# List Deployments
```
kubectl get deploy
```

# List Pods
```
kubectl get pods
kubectl get pods -o wide
```
Observation: make a note of Pod IP

# List Services
```
kubectl get svc
```
Observation: 
1. "CLUSTER-IP" will be "NONE" for Headless Service

## Sample 
```
murgod@cloudshell:~/GKE-headless-services$ kubectl get svc
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes                 ClusterIP   34.118.224.1     <none>        443/TCP    14d
my-app-clusterip-service   ClusterIP   34.118.231.209   <none>        80/TCP     46m
my-app-headless-service    ClusterIP   None             <none>        8080/TCP   39m
murgod@cloudshell:~/GKE-headless-services$ 
```
# Step-06: Review Curl Kubernetes Manifests

This step focuses on reviewing a simple Kubernetes Pod manifest for running a `curl` container. The `curl` container will be used for making HTTP requests or other testing within the Kubernetes cluster.

### Kubernetes Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl-pod
spec:
  containers:
  - name: curl
    image: curlimages/curl 
    command: [ "sleep", "600" ]
```
# Step-07: Deploy Curl-pod and Verify ClusterIP and Headless Services

## Deploy curl-pod

```bash
kubectl apply -f curl-pod.yaml
```

## List Services
```
kubectl get svc
```
## GKE Cluster Kubernetes Service Full DNS Name Format
```
<svc>.<ns>.svc.cluster.local
```
## Open a Terminal Session into the Container
```
kubectl exec -it curl-pod -- sh
```
## ClusterIP Service: nslookup and curl Test
```
nslookup my-app-clusterip-service.default.svc.cluster.local
curl my-app-clusterip-service.default.svc.cluster.local
```
## ClusterIP Service nslookup Output
```
murgod@cloudshell:~/GKE-headless-services$ kubectl exec -it curl-pod -- sh
~ $ nslookup my-app-clusterip-service.default.svc.cluster.local
Server:         169.254.20.10
Address:        169.254.20.10:53


Non-authoritative answer:
Name:   my-app-clusterip-service.default.svc.cluster.local
Address: 34.118.231.209

~ $ curl my-app-clusterip-service.default.svc.cluster.local
Hello, world!
Version: 2.0.0
Hostname: my-app-deployment-8694dc57-8n2wm
```

## Headless Service: nslookup and curl Test
```
nslookup my-app-headless-service.default.svc.cluster.local
curl my-app-headless-service.default.svc.cluster.local:8080
```
## Headless Service nslookup Output
```
~ $ nslookup my-app-headless-service.default.svc.cluster.local
Server:         169.254.20.10
Address:        169.254.20.10:53


Non-authoritative answer:
Name:   my-app-headless-service.default.svc.cluster.local
Address: 10.104.2.29
Name:   my-app-headless-service.default.svc.cluster.local
Address: 10.104.2.26
Name:   my-app-headless-service.default.svc.cluster.local
Address: 10.104.2.27
Name:   my-app-headless-service.default.svc.cluster.local
Address: 10.104.2.28

~ $ curl my-app-headless-service.default.svc.cluster.local:8080
Hello, world!
Version: 2.0.0
Hostname: my-app-deployment-8694dc57-ttzc8
```
