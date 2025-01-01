# GKE-headless-services
This repository is designed to demonstrate the practical application of Headless Services within a Google Kubernetes Engine (GKE) environment. It provides example configurations and deployments showcasing how Headless Services enable direct pod addressing. 

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GCP Google Kubernetes Engine GKE Headless Service</title>
</head>
<body>

    <h1>GCP Google Kubernetes Engine GKE Headless Service</h1>
    <p><strong>Implement GCP Google Kubernetes Engine GKE Headless Service</strong></p>

    <h2>Step-00: Pre-requisites</h2>
    <p>Verify if GKE Cluster is created</p>
    <p>Verify if kubeconfig for kubectl is configured in your local terminal</p>
    
    <h3># Configure kubeconfig for kubectl</h3>
    <pre><code>gcloud container clusters get-credentials &lt;CLUSTER-NAME&gt; --region &lt;REGION&gt; --project &lt;PROJECT&gt;</code></pre>

    <h3># Replace Values CLUSTER-NAME, ZONE, PROJECT</h3>
    <pre><code>gcloud container clusters get-credentials standard-public-cluster-1 --region us-central1 --project kdaida123</code></pre>

    <h3># List GKE Kubernetes Worker Nodes</h3>
    <pre><code>kubectl get nodes</code></pre>

    <h2>Step-01: Introduction</h2>
    <ul>
        <li>Implement Kubernetes ClusterIP and Headless Service</li>
        <li>Understand Headless Service in detail</li>
    </ul>

    <h2>Step-02: 01-kubernetes-deployment.yaml</h2>
    <pre><code>apiVersion: apps/v1
kind: Deployment
metadata: #Dictionary
  name: myapp1-deployment
spec: # Dictionary
  replicas: 4
  selector:
    matchLabels:
      app: myapp1
  template:
    metadata: # Dictionary
      name: myapp1-pod
      labels: # Dictionary
        app: myapp1  # Key value pairs
    spec:
      containers: # List
        - name: myapp1-container
          #image: stacksimplify/kubenginx:1.0.0
          image: us-docker.pkg.dev/google-samples/containers/gke/hello-app:2.0
          ports: 
            - containerPort: 8080</code></pre>

    <h2>Step-03: 02-kubernetes-clusterip-service.yaml</h2>
    <pre><code>apiVersion: v1
kind: Service
metadata:
  name: myapp1-cip-service
spec:
  type: ClusterIP # ClusterIP, # NodePort, # LoadBalancer, # ExternalName
  selector:
    app: myapp1
  ports:
    - name: http
      port: 80 # Service Port
      targetPort: 8080 # Container Port</code></pre>

    <h2>Step-04: 03-kubernetes-headless-service.yaml</h2>
    <p>Add <strong>spec.clusterIP: None</strong></p>
    <p><strong>VERY IMPORTANT NOTE</strong></p>
    <ul>
        <li>When using Headless Service, we should use both the "Service Port and Target Port" same.</li>
        <li>Headless Service directly sends traffic to Pod with Pod IP and Container Port.</li>
        <li>DNS resolution directly happens from headless service to Pod IP.</li>
    </ul>
    <pre><code>apiVersion: v1
kind: Service
metadata:
  name: myapp1-headless-service
spec:
  #type: ClusterIP # ClusterIP, # NodePort, # LoadBalancer, # ExternalName
  clusterIP: None
  selector:
    app: myapp1
  ports:
    - name: http
      port: 8080 # Service Port
      targetPort: 8080 # Container Port</code></pre>

    <h2>Step-05: Deploy Kubernetes Manifests</h2>
    <pre><code># Deploy Kubernetes Manifests
kubectl apply -f 01-kube-manifests

# List Deployments
kubectl get deploy

# List Pods
kubectl get pods
kubectl get pods -o wide

# Observation: make a note of Pod IP

# List Services
kubectl get svc
Observation: 
1. "CLUSTER-IP" will be "NONE" for Headless Service</code></pre>

    <h3>Sample Output:</h3>
    <pre><code>Kalyans-Mac-mini:19-GKE-Headless-Service kalyanreddy$ kubectl get svc
NAME                      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes                ClusterIP   10.24.0.1    &lt;none&gt;        443/TCP   135m
myapp1-cip-service        ClusterIP   10.24.2.34   &lt;none&gt;        80/TCP    4m9s
myapp1-headless-service   ClusterIP   None         &lt;none&gt;        80/TCP    4m9s
Kalyans-Mac-mini:19-GKE-Headless-Service kalyanreddy$</code></pre>

    <h2>Step-06: Review Curl Kubernetes Manifests</h2>
    <p>Project Folder: 02-kube-manifests-curl</p>
    <pre><code>apiVersion: v1
kind: Pod
metadata:
  name: curl-pod
spec:
  containers:
  - name: curl
    image: curlimages/curl 
    command: [ "sleep", "600" ]</code></pre>

    <h2>Step-07: Deploy Curl-pod and Verify ClusterIP and Headless Services</h2>
    <pre><code># Deploy curl-pod
kubectl apply -f 02-kube-manifests-curl

# List Services
kubectl get svc

# GKE Cluster Kubernetes Service Full DNS Name format
&lt;svc&gt;.&lt;ns&gt;.svc.cluster.local

# Will open up a terminal session into the container
kubectl exec -it curl-pod -- sh

# ClusterIP Service: nslookup and curl Test
nslookup myapp1-cip-service.default.svc.cluster.local
curl myapp1-cip-service.default.svc.cluster.local</code></pre>

    <h3>ClusterIP Service nslookup Output:</h3>
    <pre><code>$ nslookup myapp1-cip-service.default.svc.cluster.local
Server:		10.24.0.10
Address:	10.24.0.10:53

Name:	myapp1-cip-service.default.svc.cluster.local
Address: 10.24.2.34</code></pre>

    <h3>Headless Service: nslookup and curl Test</h3>
    <pre><code>nslookup myapp1-headless-service.default.svc.cluster.local
curl myapp1-headless-service.default.svc.cluster.local:8080</code></pre>
    
    <h3>Headless Service nslookup Output:</h3>
    <pre><code>$ nslookup myapp1-headless-service.default.svc.cluster.local
Server:		10.24.0.10
Address:	10.24.0.10:53

Name:	myapp1-headless-service.default.svc.cluster.local
Address: 10.20.0.25
Name:	myapp1-headless-service.default.svc.cluster.local
Address: 10.20.0.26
Name:	myapp1-headless-service.default.svc.cluster.local
Address: 10.20.1.28
Name:	myapp1-headless-service.default.svc.cluster.local
Address: 10.20.1.29</code></pre>

    <h2>Step-08: Clean-Up</h2>
    <pre><code># Delete Kubernetes Resources
kubectl delete -f 01-kube-manifests

# Delete Kubernetes Resources - Curl Pod
kubectl delete -f 02-kube-manifests-curl</code></pre>

</body>
</html>
