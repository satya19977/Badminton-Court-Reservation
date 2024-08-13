## Why this Project?  

### I love playing badminton. So i was trying to book a court on a busy day and realized that the process for booking a court was manual. Player would call the management and they would just use a book to record the details.  I wanted to see how i could use kubernetes to implement a solution for the problem. 
#### ![k8 Components drawio(4)](https://github.com/satya19977/Deploy-MERN-Application-Using-Kubernetes/assets/108000447/52a12216-7cc9-486e-a8d8-0f1ce519856f)

#### Internal Service: Used for communication between Pods within the same cluster.
#### External Service: Exposes a service to external traffic from outside the cluster.
#### ConfigMap: Stores configuration data as key-value pairs for consumption by Pods.
#### Secret: Stores sensitive data like passwords and keys, also consumed by Pods.
#### Deployment: Manages the deployment, scaling, and updating of applications in the cluster

## ![k8 Request_Flow drawio](https://github.com/satya19977/Deploy-MERN-Application-Using-Kubernetes/assets/108000447/5784e744-5775-45a0-bf70-0128b15a1095)
## Step 1 : Deploy Mongo Express:
#### 1.1 Create a Kubernetes Deployment manifest for Mongo Express, which will provide a web-based interface to manage MongoDB.
```
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: mongo-express
        image: mongo-express:latest
        env:
          - name: ME_CONFIG_MONGODB_ADMINUSERNAME
            valueFrom:
              secretKeyRef:
                name: mongo-secret
                key: mongo_user
          - name: ME_CONFIG_MONGODB_ADMINPASSWORD
            valueFrom:
              secretKeyRef:
                name: mongo-secret
                key: mongo_password

          - name: ME_CONFIG_MONGODB_SERVER
            valueFrom:
              configMapKeyRef:
                name: mongo-config
                key: mongo-url
        ports:
        - containerPort: 8081
```

        
#### 1.2 Expose the Mongo Express Deployment using a Kubernetes Service with type NodePort

```
apiVersion: v1
kind: Service
metadata:
  name: webapp-service 
spec:
  type: NodePort
  selector:
    app:  webapp
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30002
```
 
## Step 2 : Set Up MongoDB Cluster and Service
 #### 2.1 Create a Kubernetes Deployment manifest for MongoDB
 
 ```
 apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
  labels:
    app: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:latest
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo_user
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: mongo_password
        ports:
        - containerPort: 27017
 ```
 #### 2.2 Create a Kubernetes Service for MongoDB to expose it within the cluster.
 ```
 apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  selector:
    app: mongo
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
 ```

## Step 3 : Configure Environment Variables
#### 3.1 Define environment variables for your  Pods to connect to MongoDB.
```
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
type: Opaque
data:
   mongo_user: bW9uZ291c2Vy
   mongo_password: bW9uZ29wYXNzd29yZA==
```
#### 3.2 Configure environment variables for the Mongo Express Pod to connect to the internal MongoDB service.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-config
data: 
  mongo-url: mongo-service
```

## Step 4 : Access Your App and MongoDB
#### 4.2 Access the Mongo Express interface using the NodePort

![Screenshot (1294)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/f60a8872-6216-4a15-85c5-cb5680d0d5be)



# Output


![Screenshot (1293)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/826e72a1-bef6-4adf-8cfe-372b3c57ff1c)


