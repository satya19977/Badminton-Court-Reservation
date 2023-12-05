# We want to use an ALB ingress controller to expose our application outside the cluster.


### Download the policy

```
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
### Create an IAM policy
```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy2 --policy-document file://iam_policy.json
```
### Create an iam serviceaccount
```
eksctl create iamserviceaccount  --cluster=ultimate-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole2 --attach-policy-arn=arn:aws:iam::61625:policy/AWSLoadBalancerControllerIAMPolicy2 --approve --override-existing-serviceaccounts
```
### Associate your OIDC to your cluster 
```
eksctl utils associate-iam-oidc-provider --cluster ultimate-cluster --approve
```
### Download Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
### Add helm repo
```
helm repo add eks https://aws.github.io/eks-charts
```
### Update repo
```
helm repo update eks
```
```
helm install aws-load-balancer eks/aws-load-balancer-controller --namespace kube-system   --set clusterName=ultimate-cluster   --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1  --set vpcId=vpc-0e0051d40dc41
``` 

## Ingress Resource for Web-application
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-app
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: instance
    alb.ingress.kubernetes.io/group.name: apg  # Choose a unique name for your group

spec:
  rules:
    - host: "cluster.myprojectssatya.click"
      http:
        paths:
          - path: "/"
            pathType: Prefix
            backend:
              service:
                name: webapp-service 
                port:
                  number: 8083
```
### After you have completed the above steps


### Copy the address and paste it in your browser

# ![Screenshot (1527)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/1796b103-969a-4c16-a3e6-491cdef0dd5f)



## Ingress Resource for Prometheus

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-prometheus
  namespace: prometheus
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: instance
    alb.ingress.kubernetes.io/group.name: apg  # Choose a unique name for your group
spec:
  rules:
    - host: "prometheus.myprojectssatya.click"
      http:
        paths:
          - path: "/"
            pathType: Prefix
            backend:
              service:
                name: prometheus-server-out 
                port:
                  number: 85
```
# ![Screenshot (1525)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/6b2c46a5-319b-4917-9613-347723a3d053)

## Ingress Resource for Grafana

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-grafana
  namespace: grafana
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: instance
    alb.ingress.kubernetes.io/group.name: apg
spec:
  rules:
    - host: "grafana.myprojectssatya.click"
      http:
        paths:
          - path: "/"
            pathType: Prefix
            backend:
              service:
                name: grafana-out
                port:
                  number: 8084
```

# ![Screenshot (1526)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/193a0f5c-0fef-4830-a8e3-4bbc3275aa39)

