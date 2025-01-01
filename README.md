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
