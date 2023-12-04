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

### Configuration to deploy Karpenter
```

########################################################
# How To Auto-Scale Kubernetes Clusters With Karpenter #
# https://youtu.be/C-2v7HT-uSA                         #
########################################################

# Referenced videos:
# - Karpenter: https://karpenter.sh
# - GKE Autopilot - Fully Managed Kubernetes Service From Google: https://youtu.be/Zztufl4mFQ4

#########
# Setup #
#########

git clone https://github.com/vfarcic/karpenter-demo

cd karpenter-demo

export CLUSTER_NAME=devops-toolkit

# Replace `[...]` with your access key ID
export AWS_ACCESS_KEY_ID=[...]

# Replace `[...]` with your secret access key
export AWS_SECRET_ACCESS_KEY=[...]

export AWS_DEFAULT_REGION=us-east-1

cat cluster.yaml \
    | sed -e "s@name: .*@name: $CLUSTER_NAME@g" \
    | sed -e "s@region: .*@region: $AWS_DEFAULT_REGION@g" \
    | tee cluster.yaml

cat provisioner.yaml \
    | sed -e "s@instanceProfile: .*@instanceProfile: KarpenterNodeInstanceProfile-$CLUSTER_NAME@g" \
    | sed -e "s@.* # Zones@      values: [${AWS_DEFAULT_REGION}a, ${AWS_DEFAULT_REGION}b, ${AWS_DEFAULT_REGION}c] # Zones@g" \
    | tee provisioner.yaml

cat app.yaml \
    | sed -e "s@topology.kubernetes.io/zone: .*@topology.kubernetes.io/zone: ${AWS_DEFAULT_REGION}a@g" \
    | tee app.yaml

eksctl create cluster \
    --config-file cluster.yaml

export CLUSTER_ENDPOINT=$(aws eks describe-cluster \
    --name $CLUSTER_NAME \
    --query "cluster.endpoint" \
    --output json)

echo $CLUSTER_ENDPOINT

###########################
# Karpenter Prerequisites #
###########################

export SUBNET_IDS=$(\
    aws cloudformation describe-stacks \
    --stack-name eksctl-$CLUSTER_NAME-cluster \
    --query 'Stacks[].Outputs[?OutputKey==`SubnetsPrivate`].OutputValue' \
    --output text)

aws ec2 create-tags \
    --resources $(echo $SUBNET_IDS | tr ',' '\n') \
    --tags Key="kubernetes.io/cluster/$CLUSTER_NAME",Value=

curl -fsSL https://raw.githubusercontent.com/aws/karpenter/v0.30.0/website/content/en/preview/getting-started/getting-started-with-karpenter/cloudformation.yaml \
    | tee karpenter.yaml

aws cloudformation deploy \
    --stack-name Karpenter-$CLUSTER_NAME \
    --template-file karpenter.yaml \
    --capabilities CAPABILITY_NAMED_IAM \
    --parameter-overrides ClusterName=$CLUSTER_NAME

export AWS_ACCOUNT_ID=$(\
    aws sts get-caller-identity \
    --query Account \
    --output text)

eksctl create iamidentitymapping \
    --username system:node:{{EC2PrivateDNSName}} \
    --cluster  $CLUSTER_NAME \
    --arn arn:aws:iam::${AWS_ACCOUNT_ID}:role/KarpenterNodeRole-$CLUSTER_NAME \
    --group system:bootstrappers \
    --group system:nodes

eksctl create iamserviceaccount \
    --cluster $CLUSTER_NAME \
    --name karpenter \
    --namespace karpenter \
    --attach-policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/KarpenterControllerPolicy-$CLUSTER_NAME \
    --approve

# Execute only if this is the first time using spot instances in this account
aws iam create-service-linked-role \
    --aws-service-name spot.amazonaws.com
```

### Yaml for karpenter is given below

```
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: karpenter-provisioner
spec:
  limits:
   resources:
    cpu: 2000
  requirements:
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["spot"]
  ttlSecondsAfterEmpty: 20
  provider:
    instanceProfile: arn:aws:iam::69@#$$%:role/KarpenterNodeRole-ultimate-cluster
```
This will look for spot instances to spin up which have Vcpus less than  2. 





