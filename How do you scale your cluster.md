# Scaling is an important part of your cluster as it enables you to serve traffic without any hiccups!!

## HorizontalPodAutoscaler to scale your pod 

We want to scale our deployment named webapp-deployment based on CPU utilizations of > 60% 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  namespace: events-cluster 
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
        resources:
          requests:
            cpu: 100m  # Set your desired CPU request here
            memory: 128Mi 
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

        
---
# Mwebapp-Service
apiVersion: v1
kind: Service
metadata:
  name: webapp-service 
  namespace: events-cluster 
spec:
  type: NodePort
  selector:
    app:  webapp
  ports:
    - protocol: TCP
      port: 8083
      targetPort: 8081
      nodePort: 30002
```
## Yaml file for HPA

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-scaling
  namespace: events-cluster
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```
