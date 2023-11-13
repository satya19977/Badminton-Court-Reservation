We want to use an ALB ingress controller to expose our application outside the cluster.

## Prerequisites
Download the policy
```
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
Create an IAM policy
```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy2 --policy-document file://iam_policy.json
```
Create an iam serviceaccount
```
eksctl create iamserviceaccount  --cluster=ultimate-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole2 --attach-policy-arn=arn:aws:iam::61625:policy/AWSLoadBalancerControllerIAMPolicy2 --approve --override-existing-serviceaccounts
```
Associate your OIDC to your cluster 
```
eksctl utils associate-iam-oidc-provider --cluster ultimate-cluster --approve
```
Download Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
Add helm repo
```
helm repo add eks https://aws.github.io/eks-charts
```
Update repo
```
helm repo update eks
```
```
helm install aws-load-balancer eks/aws-load-balancer-controller --namespace kube-system   --set clusterName=ultimate-cluster   --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1  --set vpcId=vpc-0e0051d40dc41
``` 

## Yaml for Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: events-cluster 
  name: em2
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: webapp-service 
              port:
                number: 8083
```
After you have completed the above steps

```
Kubectl get ingress
```
Copy the address and paste it in your browser
