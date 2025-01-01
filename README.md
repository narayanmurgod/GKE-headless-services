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

     Example:
     ```bash
     gcloud container clusters get-credentials standard-public-cluster-1 --region us-central1 --project kdaida123
     ```

3. **List GKE Kubernetes Worker Nodes**:
   Verify that your nodes are running correctly.

   ```bash
   kubectl get nodes

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

