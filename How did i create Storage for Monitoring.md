## Why are we creating storage for this cluster? The prometheus server downloaded using helm doesn't automatically create a storage solution i.e it doesn't dynamically provision a storage for us.

### Prerequisites
1. Have an EKS Cluster up and running.



#### Associate OIDC to connect kuberntes cluster to your AWS Account

```
eksctl utils associate-iam-oidc-provider --cluster ultimate-cluster  --approve
```
Create a Trust Policy
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::616149:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/C9AA8B2A"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.us-east-1.amazonaws.com/id/C9AADAA4A8B2A:aud": "sts.amazonaws.com",
          "oidc.eks.us-east-1.amazonaws.com/id/C9AAD6A8B38B2A:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa"
        }
      }
    }
  ]
} 
```
#### Create an iam role
```
aws iam create-role --role-name AmazonEKS_EBS_CSI_Driver --assume-role-policy-document file://"trust-policy.json"
```
#### Create iam policy 
```
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --role-name AmazonEKS_EBS_CSI_Driver
```
#### Deploy the role
```
aws eks create-addon --cluster-name ultimate-cluster --addon-name aws-ebs-csi-driver --service-account-role-arn arn:aws:iam::616251831149:role/AmazonEKS_EBS_CSI_Driver
```
```
eksctl get addon --cluster ultimate-cluster 

```
#### Annotate Service Accounts of promtheus-server and prometheus-alert-manager 

```
arn:aws:iam::<account-id>:role/rolename
```


# ![Screenshot (1462)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/1b8af8b0-ca33-48cb-baba-0bfbf5693f24)










