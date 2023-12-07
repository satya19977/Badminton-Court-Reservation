## For monitoring we use prometheus which scrapes metrics data and to visualize our findings we use grafana


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



#### You can take the address of each ingress resource and map it to a domain name to create something like grafana.something.com or prometheus. 

![Screenshot (1474)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/d713149a-10fb-42bd-9c07-2c41dc97aa64)

### A Sample of our Grafana Dashboard which exposes metrics from metrics server endpoint

![Screenshot (1501)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/434744ce-e6ad-4f28-8339-89691241995a)

![Screenshot (1500)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/0b636bcb-6d64-4a7d-ac3d-40066e4a7b7d)

### The above dashboard is a basic one. If we want to expose advanced  metrics we need to use kube-state-metrics.
1. To do that we need to expose our kube-state-metrics pod to nodeport

2. Get the Configmap of the prometheus-server and change the targethost to whatever is required
![Screenshot (1503)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/e289b2d5-3b7b-438f-9433-5ad4a8a27085)



   