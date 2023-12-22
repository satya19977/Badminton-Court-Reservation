## For monitoring we use prometheus which scrapes our desired target on the /metrics endpoint and to visualize our findings we use grafana


### Node Metrics 

![Screenshot (1546)](https://github.com/satya19977/Badminton-Court-Reservation/assets/108000447/8cfdde7a-a8df-4e5e-aed4-75bf8cbfeeda)


### Cluster Metrics

![Screenshot (1547)](https://github.com/satya19977/Badminton-Court-Reservation/assets/108000447/79cfbb7b-e6e8-4f85-9dc4-fc87c486b38c)



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





   
