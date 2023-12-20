## For monitoring we use prometheus which scrapes our desired target on the /metrics endpoint and to visualize our findings we use grafana


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





### We pulled basic metrics such as DiskSpace used, CPU utilization, information such as number of cores, RAM on the system


![Screenshot (1546)](https://github.com/satya19977/Badminton-Court-Reservation/assets/108000447/8cfdde7a-a8df-4e5e-aed4-75bf8cbfeeda)



### The above dashboard exposes metrics about the node. If we want to expose cluster metrics we need to use kube-state-metrics. 
1. To do that we need to expose our kube-state-metrics pod to nodeport

```
kubectl expose service prometheus-kube-state-metrics  --type=NodePort  --target-port=8080  --name=kube-state-out -n prometheus
```
 

2. Get the Configmap of the prometheus-server and change the targethost to whatever is required
![Screenshot (1503)](https://github.com/satya19977/Kubernetes-Event-Operations/assets/108000447/e289b2d5-3b7b-438f-9433-5ad4a8a27085)

3. Go to your targets in the prometheus dropdown and select targets and confirm if your endpoint is there
![Screenshot (1544)](https://github.com/satya19977/Badminton-Court-Reservation/assets/108000447/a1802c2d-3da4-4ea9-9a9f-378603a9fe03)

### Alerting Rules

This is a simple rule that fires an alert whenever our nodes are down
```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    release: prometheus
  name: node-down
spec:
  groups:
  - name: instancedown
    rules:
    - alert: NodeDown
      expr: up == 0
      for: 0m
      labels:
        severity: critical
      annotations:
        summary: "Node Down Alert"
        description: "A node is unavailable (down)"
```

![Screenshot (1545)](https://github.com/satya19977/Badminton-Court-Reservation/assets/108000447/c78c1356-f655-4937-8406-b44688bf7597)


### Test
#### 1.Node CPU Usage: This check what is the idle time of our Node CPU. We want it to be low




   
