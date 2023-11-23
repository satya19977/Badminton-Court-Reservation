# For monitoring we use prometheus which scrapes metrics data and to visualize our findings we use grafana


## Prerequisites 
1.Metrics Server


### Helm chart for Prometheus

### Add Helm repo
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
### Update repo

```
helm repo update
```
### Install helm

```
helm install prometheus prometheus-community/prometheus
```
### Expose the service 

```
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-out
```
### Helm Chart for Grafana

install grafana

### Add Helm repo
```
helm repo add grafana https://grafana.github.io/helm-charts
```
### Update repo

```
helm repo update
```
### Install helm

```
helm install grafana grafana/grafana
```
### Expose the service 

```
kubectl expose service grafana  --type=NodePort  --target-port=3000  --name=grafana-out -n grafana
```
### Create an ingress Resource for prometheus 

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: em2-prometheus
  namespace: prometheus
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
  
   - http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: prometheus-server-out
            port:
              number: 85
```
### Create an ingress resource for grafana

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: grafana
  name: em2-grafana
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
              name: grafana-out
              port:
                number: 8084
```
# ![Screenshot (1473)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/aebb5ba9-5843-471a-af73-1414cf200f01)

#### You can take the address of each ingress resource and map it to a domain name to create something like grafana.something.com or prometheus. 

![Screenshot (1474)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/d713149a-10fb-42bd-9c07-2c41dc97aa64)

### A Sample of our Grafana Dashboard which exposes metrics from metrics server endpoint

![Screenshot (1501)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/434744ce-e6ad-4f28-8339-89691241995a)

![Screenshot (1500)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/0b636bcb-6d64-4a7d-ac3d-40066e4a7b7d)

