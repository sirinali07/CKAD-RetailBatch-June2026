# Prometheus Monitoring on Kubernetes using Helm

## Business Scenario

Your organization is running containerized applications on Kubernetes.

The Platform Engineering team requires a monitoring solution capable of collecting:

* Node Metrics
* Pod Metrics
* Kubernetes Object Metrics
* Resource Utilization Metrics

As a Kubernetes Administrator, your task is to deploy Prometheus and validate monitoring functionality.

---

## What is Prometheus?

Prometheus is an open-source monitoring and alerting platform maintained by the Cloud Native Computing Foundation (CNCF).

Prometheus collects metrics from applications and infrastructure and stores them as time-series data.

Common use cases include:

* Infrastructure Monitoring
* Kubernetes Monitoring
* Application Monitoring
* Capacity Planning
* Alerting and Incident Detection

---

## Prometheus Architecture

```text
+------------------------------------------------+
|                Prometheus Server               |
+------------------------------------------------+
                     |
                     |
        --------------------------------
        |              |              |
        |              |              |
        v              v              v

Node Exporter  Kube-State-Metrics  Applications

                     |
                     |
                     v

               Alertmanager

                     |
                     |
                     v

           Email / Teams / Slack
```

---

## Prometheus Components

### Prometheus Server

Responsible for:

* Collecting metrics
* Storing metrics
* Running queries
* Evaluating alerts

### Node Exporter

Collects infrastructure metrics such as:

* CPU Usage
* Memory Usage
* Disk Usage
* Network Statistics

### Kube-State-Metrics

Collects Kubernetes object information such as:

* Pods
* Deployments
* Nodes
* StatefulSets
* Services


### Alertmanager

Processes alerts and sends notifications to:

* Email
* Microsoft Teams
* Slack
* PagerDuty

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
         +-----------------------------+
         |      Prometheus Server      |
         +-----------------------------+
                       |
      ----------------------------------------
      |                  |                  |
      |                  |                  |
      v                  v                  v

Node Exporter   Kube-State-Metrics   Applications

                       |
                       |
                       v

                 Alertmanager

                       |
                       |
                       v

                LoadBalancer

                       |
                       |
                       v

             Browser Access
```

---

## Prerequisites

Before starting the lab, ensure:

* Kubernetes Cluster is available
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

Expected Output:

```text
Version:"v3.x.x"
```

### Validation

Helm should return a valid version.

---

## Task 2 - Add Prometheus Helm Repository


Helm repositories host application charts.

Add repository:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Update repository metadata.

```bash
helm repo update
```

Verify repository.

```bash
helm search repo prometheus
```

Expected Output:

```text
prometheus-community/prometheus
```

### Validation

Prometheus chart should be visible.

---

## Task 3 - Create Monitoring Namespace

Create a dedicated namespace.

```bash
kubectl create namespace monitoring
```

Verify:

```bash
kubectl get ns
```

Expected Output:

```text
monitoring   Active
```

| **Note :** Namespaces isolate workloads and simplify management.

---

## Task 4 - Install Prometheus

Deploy Prometheus using Helm.

```bash
helm install prometheus prometheus-community/prometheus \
--namespace monitoring \
--set server.service.type=LoadBalancer
```

 | ***What happens behind the scenes?**
 | Helm automatically creates:
  | * Deployments
 | * Services
 | * ConfigMaps
 | * Service Accounts
 | * RBAC Rules
 | * Persistent Storage



Verify installation.

```bash
helm list -n monitoring
```
| Status should display deployed



---

## Task 5 - Verify Pods

Check Prometheus components.

```bash
kubectl get pods -n monitoring
```

| All pods should show Running


---

## Task 6 - Verify Services

Check services.

```bash
kubectl get svc -n monitoring
```

|  The cloud provider creates a public IP and routes traffic to Prometheus.

---

## Task 7 - Access Prometheus

Monitor service creation.

```bash
kubectl get svc prometheus-server -n monitoring 
```

Example:
| http://20.198.45.120
| OR
| ab79c9763f25f4fd9b244f9caad6b8aa-316764806.ca-central-1.elb.amazonaws.com

Expected Result:

Prometheus Dashboard loads successfully.

---

## Task 8 - Verify Targets

Navigate to: ***Status → Targets***


**Validation**

All targets should be healthy.

---

## Task 9 - Execute PromQL Queries

Navigate to: **Graph**

*** Query 1 - Target Health ***

*** Query 2 - Total Pods ***

*** Query 3 - Total Nodes ***

*** Query 4 - Available Memory ***

*** Query 5 - CPU Usage Trend ***

---

---

## Common Troubleshooting

#### Pods Not Starting

Check:

```bash
kubectl get pods -n monitoring
kubectl describe pod <pod-name> -n monitoring
```

#### LoadBalancer Stuck in Pending

Check:

```bash
kubectl describe svc prometheus-server \
-n monitoring
```

Possible Causes:

* Cloud quota exhausted
* Unsupported environment
* Missing Load Balancer integration

#### Prometheus Not Accessible

Verify:

```bash
kubectl get svc -n monitoring
```

Ensure External IP is assigned.

---

## Cleanup

Remove Prometheus.

```bash
helm uninstall prometheus -n monitoring
```

Delete namespace.

```bash
kubectl delete namespace monitoring
```

Verify:

```bash
kubectl get ns
```

---

## Lab Summary

Congratulations!


---

## Additional Challenge

Try the following:

1. Install Grafana using Helm.
2. Connect Grafana to Prometheus.
3. Import Kubernetes Monitoring Dashboard.
4. Create a CPU Utilization Dashboard.

