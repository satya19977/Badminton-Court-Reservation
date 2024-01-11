## Scaling is an important part of your cluster as it enables you to serve traffic without any hiccups and make your infrastructure reliable !!
There are two types of Scaling
1. Kubernetes Cluster Scaling Using HPA
2. Node Scaling Using Karpenter

## Prerequisites
### Install Kubernetes metrics server in EKS cluster
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
### Check if metrics server has been deployed
```
Kubectl get deploy metrics-server -n kube-system
```


## HorizontalPodAutoscaler to scale your pod

### We want to scale our deployment named webapp-deployment based on CPU utilizations of > 60% 

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
### This is the output before load was generated 

![Screenshot (1449)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/dbe4ef1d-c84f-4a30-8958-082ba5ef4ad6)

As is evident from above we see that only one pod is active

### Generate Load
```
kubectl run -i --tty load-generator4 --rm --image=alpine --restart=Never -- /bin/sh
apk --no-cache add curl
while sleep 0.01; do curl -s -u admin:pass http://k8s-eventscl-em2-8d19ca2457-405296923.us-east-1.elb.amazonaws.com; done
```

![Screenshot (1450)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/885350d8-089b-4fce-af2b-b80ea0441e0a)

The HPA kicked in and the number of pods has scaled upto 5 which was the limit set in our yaml file

## We used HPA to scale pod traffic but what if the pods reach node limit. In that case we need to scale up our nodes as well. That's where Karpenter comes in. 

#What is Karpenter
#Why karpenter over Cluster Auto-Scaler
#How does Karpenter work

### Test1. Auto-Scaling and time taken 

Provisioner.yaml
```
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]
        - key: kubernetes.io/os
          operator: In
          values: ["linux"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot"]
        - key: karpenter.k8s.aws/instance-category
          operator: In
          values: ["c", "m", "r"]
        - key: karpenter.k8s.aws/instance-generation
          operator: Gt
          values: ["2"]
      nodeClassRef:
        name: default
  limits:
    cpu: 8000
  disruption:
    consolidationPolicy: WhenUnderutilized
    expireAfter: 1h 
---
apiVersion: karpenter.k8s.aws/v1beta1
kind: EC2NodeClass
metadata:
  name: default
spec:
  amiFamily: AL2 # Amazon Linux 2
  role: "KarpenterNodeRole-ultimate-cluster" 
  subnetSelectorTerms:
    - tags:
        karpenter.sh/discovery: "ultimate-cluster" 
  securityGroupSelectorTerms:
    - tags:
        karpenter.sh/discovery: "ultimate-cluster" 
```

![Screenshot (1536)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/9f35ba80-7b28-4cb9-b31a-707d89e5f430)
### We are running two nodes of type  m5.large
![Screenshot (1535)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/a2b5f9e7-ee16-42d1-b527-bf73fcb6bf2a)



 Sample Deployment  that generates load
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inflate
spec:
  replicas: 0
  selector:
    matchLabels:
      app: inflate
  template:
    metadata:
      labels:
        app: inflate
    spec:
      terminationGracePeriodSeconds: 0
      containers:
        - name: inflate
          image: public.ecr.aws/eks-distro/kubernetes/pause:3.7
          resources:
            requests:
              cpu: 1
```
![Screenshot (1538)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/ae86c6bf-f22a-474b-915c-d892089bc09b)

### We introduced load and karpenter spun up a new spot instance in about 45 seconds
![Screenshot (1539)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/58f3173f-6f9f-480b-a1a1-6e5203d6b4fa)

### Test2. Limit set on vcpu

This pod has memory set to 3000m but the limit in our yaml file is 2000m
```
apiVersion: v1
kind: Pod
metadata:
  
  labels:
    run: tmp
  name: tmp
spec:
  containers:
  - image: nginx
    name: tmp
    resources: 
      requests:
       memory: "64Mi"
       cpu: "3000m"
       
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

![Screenshot (1540)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/dd6e3882-81a8-419a-916e-6b29e0cf47fb)

### The pod doesn't get scheduled and karpenter logs displays the error saying not comapatible with the Nodepool
![Screenshot (1541)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/53ddbc79-21de-4edc-a155-0ede14d9fadd)






