# Grafana Dashboarding on Kubernetes using Helm

## Business Scenario

Your organization has already implemented Prometheus for collecting Kubernetes and infrastructure metrics.

The Operations team now requires a centralized dashboarding solution to:

* Visualize Kubernetes metrics
* Monitor infrastructure health
* Create custom dashboards
* Analyze trends and performance
* Share dashboards with operations teams

As a Kubernetes Administrator, your task is to deploy Grafana, connect it to Prometheus, and build monitoring dashboards.

---

## What is Grafana?

Grafana is an open-source observability and visualization platform used to create dashboards from various data sources.

Grafana does not collect metrics itself. Instead, it connects to data sources such as:

* Prometheus
* Elasticsearch
* Azure Monitor
* CloudWatch
* InfluxDB
* SQL Databases

---

## Why Grafana?

While Prometheus stores metrics, Grafana provides visualization capabilities.

Common use cases include:

* Infrastructure Monitoring
* Kubernetes Monitoring
* Application Monitoring
* Business Dashboards
* Performance Analysis
* Executive Reporting

---

## Grafana Architecture

```text
+--------------------------------------+
|               Grafana                |
+--------------------------------------+
                |
                |
                v

+--------------------------------------+
|             Prometheus               |
+--------------------------------------+
                |
                |
      -------------------------
      |           |           |
      v           v           v

Node Exporter  Kube-State   Applications
               Metrics
```

---

## Grafana Components

### Dashboards

Dashboards provide visual representations of metrics.

Examples:

* CPU Utilization
* Memory Usage
* Pod Health
* Node Health
* Application Performance

### Panels

A dashboard consists of multiple panels.

Examples:

* Graph
* Gauge
* Table
* Stat
* Heatmap

### Data Sources

Data sources provide metrics to Grafana.

Common data sources:

* Prometheus
* Azure Monitor
* Elasticsearch
* Loki
* CloudWatch

### Users and Organizations

Grafana supports:

* Role-Based Access Control (RBAC)
* Multiple Users
* Teams
* Organizations

---

## Lab Architecture

```text
+----------------------------------------------------+
|                Kubernetes Cluster                  |
+----------------------------------------------------+
                       |
                       |
               Monitoring Namespace
                       |
      ------------------------------------------
      |                                        |
      |                                        |
      v                                        v

+------------------+              +------------------+
|     Grafana      |------------->|   Prometheus     |
+------------------+              +------------------+
                                            |
                                            |
                           ----------------------------------
                           |                |               |
                           v                v               v

                    Node Exporter   Kube-State-Metrics   Apps

                                            |
                                            |
                                            v

                                      Metrics Store

                       |
                       v

                 LoadBalancer

                       |
                       v

                 Browser Access
```

---

## Prerequisites

Before starting the lab, ensure:

* Kubernetes Cluster is available
* Prometheus is already deployed
* kubectl is configured
* Helm v3 is installed
* Internet connectivity is available
* Cluster Admin permissions are assigned

---

## Task 1 - Verify Helm Installation

Check Helm version.

```bash
helm version
```

### Validation

Helm should return a valid version.

---

## Task 2 - Add Grafana Helm Repository

Add repository:

```bash
helm repo add grafana https://grafana.github.io/helm-charts
```

Update repository metadata.

```bash
helm repo update
```

Verify repository.

```bash
helm search repo grafana
```

### Validation

Grafana chart should be visible.

---

## Task 3 - Verify Monitoring Namespace

Check namespace.

```bash
kubectl get ns
```

`Note` - If namespace does not exist install it (Optional):

```bash
kubectl create namespace monitoring
```

---

## Task 4 - Install Grafana

Deploy Grafana using Helm.

```bash
helm install grafana grafana/grafana \
--namespace monitoring \
--set service.type=LoadBalancer
```

### What happens behind the scenes?

| Helm automatically creates:
| * Deployment
| * Service
| * ConfigMap
| * Secret
| * Service Account
| * Persistent Storage



Verify installation.

```bash
helm list -n monitoring
```

***Validation*** 

Status should display: deployed


---

## Task 5 - Verify Grafana Pods

Check pods.

```bash
kubectl get pods -n monitoring
```


---

## Task 6 - Verify Services

Check services.

```bash
kubectl get svc -n monitoring
```

---

## Task 7 - Retrieve Grafana Admin Password

Get the default admin password.

```bash
kubectl get secret \
--namespace monitoring grafana \
-o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

---

## Task 8 - Access Grafana

Check LoadBalancer IP.

```bash
kubectl get svc grafana -n monitoring
```

Open browser.

| http://20.198.45.125 | OR | http://a6f1b8b2d12345.elb.amazonaws.com |


Login using:

| Username: admin
| Password: <retrieved-password>
 
**Validation**

Grafana Dashboard should load successfully.

---

## Task 9 - Add Prometheus Data Source

1. Navigate to:

Connections → Data Sources

2. Click:

Add Data Source


3. Select:

Prometheus

4. Enter URL:

| http://<prometheus service i.e LoadBalancer Service URL>

http://a36f2c5ec321145f89d6999ef5712ea9-1005833477.ca-central-1.elb.amazonaws.com 


5. Leave other setting as it is and click:

Save & Test


### Validation

Message should display:  `Data source is working`


---

## Task 10 - Create Dashboard

1. Navigate to:

Dashboards → New → Import

2. Import Dashboard ID:

```text
15757
```

3. Click → Load
 
4. Select Prometheus Data Source.

5. Click → Import


### Validation

Kubernetes monitoring dashboard should load successfully.

---

## Task 14 - Explore Dashboards

Review metrics:

* CPU Usage
* Memory Usage
* Node Health
* Pod Count
* Cluster Status
* Workload Performance

---

## Common Troubleshooting

### Grafana Pod Not Starting

Check:

```bash
kubectl get pods -n monitoring
kubectl describe pod <grafana-pod> -n monitoring
```

### LoadBalancer Pending

Check:

```bash
kubectl describe svc grafana -n monitoring
```

Possible Causes:

* Cloud quota exhausted
* Missing Load Balancer integration
* Unsupported Kubernetes platform


### Unable to Login

Reset password.

```bash
kubectl delete secret grafana -n monitoring
helm upgrade grafana grafana/grafana -n monitoring
```


### Prometheus Data Source Failing

Verify Prometheus Service.

```bash
kubectl get svc -n monitoring
```

Check connectivity.

```bash
kubectl exec -it <grafana-pod> -n monitoring -- sh
```

```bash
wget -qO- http://prometheus-server.monitoring.svc.cluster.local
```

---

## Cleanup

Remove Grafana.

```bash
helm uninstall grafana -n monitoring
```

Verify:

```bash
helm list -n monitoring
```

---

## Lab Summary

Congratulations!

You have successfully:


