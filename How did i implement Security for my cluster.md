## Security defines how access to cluster is given. 
By default in EKS, you cannot modify through aws-auth the permissions of the account that created the EKS cluster

### How do we secure our cluster. 
#### 1. We use RBAC
#### 2. We use Network Policies
#### 3. We use IRSA - enables application running in cluster to authenticate with AWS using IAM roles

## Prerequisites
1. Create two users in and no permissions attached to them in IAM

### 1. Give admin access to other users
![Screenshot (1483)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/f9f1f94c-04a9-473a-9c69-fb871f516296)

#### We see that the user1 has no permission to access the cluster 

#### We need to modify the aws-auth file and give admin access to the cluster to user1
```
kubectl edit -n kube-system configmap/aws-auth
```

![Screenshot (1485)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/f4776ba4-99fb-4704-bd5d-0eed31654828)

#### After modfying the above file , we see that user1 can access pods and pretty much everything in the cluster but this is not an ideal situation and we should enforce granular access using RBAC

![Screenshot (1486)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/624a86ae-81cd-4072-a496-7692d10a0610)

### 2. Control access using RBAC

#### We create another user user2 with the same IAM permissions as the first user but we use Role and RoleBinding to cut down access to our cluster

### Create a role in events-cluster namespace
```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: events-role
  namespace: events-cluster
rules:
  - apiGroups:
      - ""
      - extensions
      - apps
    resources:
      - deployments
      - replicasets
      - pods
    verbs:
      - get
      - list
      - watch
      - patch  
```
### Create a rolebinding in events-cluster namespace

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: events-rolebinding
  namespace: events-cluster
roleRef:
  apiGroup: ""
  kind: Role
  name: events-role
subjects:
  - kind: User
    name: test2-ultimate
    apiGroup: ""
```
#### As is evident our user has no access to pods and resources defined in our cluster

![Screenshot (1488)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/ee126ee1-90f4-45c1-9729-32436406ab0f)

#### 3. Use Network Policies to restrict access to mongodb. Only pods from webapp should be allowed
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mongodb-network-policy
  namespace: events-cluster
spec:
  podSelector:
    matchLabels:
      app: mongo
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: webapp
```

#### We created a temporary pod tmp in the same namesapce and tried to connect to mongopod. As is evident from the picture below our connection was refused

![Screenshot (1490)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/6ee03d6d-6249-40e4-9c2c-a847cbda18c3)

