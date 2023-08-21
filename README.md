## I love playing badminton. So i was trying to book a court on a busy day and noticed that the server crashed due to heavy traffic on the website.  I then  wanted to design a  simple but highly scalable and cost efficient architecture 
## This Architecture was designed to mimick small scale event management sys
## ![k8 Request_Flow drawio](https://github.com/satya19977/Deploy-MERN-Application-Using-Kubernetes/assets/108000447/5784e744-5775-45a0-bf70-0128b15a1095)
## Step 1 : Deploy Mongo Express:
#### 1.1 Create a Kubernetes Deployment manifest for Mongo Express, which will provide a web-based interface to manage MongoDB.
#### 1.2 Expose the Mongo Express Deployment using a Kubernetes Service with type NodePort
 
## Step 2 : Set Up MongoDB Cluster and Service
 #### 2.1 Create a Kubernetes Service for MongoDB to expose it within the cluster.

## Step 3 : Configure Environment Variables
#### 3.1 Define environment variables for your frontend and backend Pods to connect to MongoDB.
#### 3.2 Configure environment variables for the Mongo Express Pod to connect to the internal MongoDB service.

## Step 4 : Access Your App and MongoDB
#### 4.1 Access your frontend application using the NodePort or LoadBalancer IP and port.
#### 4.2 Access the Mongo Express interface using the NodePort

### ![k8 Components drawio(4)](https://github.com/satya19977/Deploy-MERN-Application-Using-Kubernetes/assets/108000447/52a12216-7cc9-486e-a8d8-0f1ce519856f)

#### Internal Service: Used for communication between Pods within the same cluster.
#### External Service: Exposes a service to external traffic from outside the cluster.
#### ConfigMap: Stores configuration data as key-value pairs for consumption by Pods.
#### Secret: Stores sensitive data like passwords and keys, also consumed by Pods.
#### Deployment: Manages the deployment, scaling, and updating of applications in the cluster
