# Lab: WordPress Frontend + MariaDB Database with NetworkPolicy

This lab demonstrates how to enforce pod-to-pod network restrictions in Kubernetes using NetworkPolicy. The goal is to allow only the WordPress frontend to reach the MariaDB database on TCP port 3306 and deny all other pods.

**Audience:** engineers learning Kubernetes network segmentation and Zero Trust microsegmentation.

---

## Architecture (logical)

Simple 2-tier application:

```
+------------------+
|    WordPress     |
|   (Frontend)     |
+--------+---------+
         |
         | TCP 3306 (allowed)
         v
+------------------+
|     MariaDB      |
|    Database      |
+------------------+

Any other Pod ---> MariaDB  ❌ (denied)
```

Note: Only WordPress (app=wordpress) may initiate TCP/3306 to the MariaDB pods (app=mariadb).

---

## Prerequisites

- A Kubernetes cluster with a network plugin that supports NetworkPolicy (Calico, Cilium, Kube-router, etc.).
- `kubectl` configured for the target cluster.

Note: This guide only documents the lab steps and verification commands; it does not change any manifests or cluster behavior.

---

## Lab Steps (with Notes)

Each step includes a short Note explaining intent and the commands/manifests used.

### Step 1 — Create namespace
Note: Isolate lab resources into a dedicated namespace for easy cleanup and clear scoping.

Command:

```
kubectl create ns wordpress-lab
```

---

### Step 2 — Deploy MariaDB
Note: Deploy a single-replica MariaDB instance pre-seeded with a `wordpress` DB and a `wpuser` account.

```
vi mariadb.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  namespace: wordpress-lab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:11
        env:
        - name: MARIADB_ROOT_PASSWORD
          value: rootpass
        - name: MARIADB_DATABASE
          value: wordpress
        - name: MARIADB_USER
          value: wpuser
        - name: MARIADB_PASSWORD
          value: wppass
        ports:
        - containerPort: 3306
```
save the file using `ESCAPE + :wq!`

Apply:

```
kubectl apply -f mariadb.yaml
```

---

### Step 3 — Create MariaDB Service
Note: Expose MariaDB internally in the cluster via a ClusterIP `Service` named `mariadb`.
```
vi mariadb-svc.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  namespace: wordpress-lab
spec:
  selector:
    app: mariadb
  ports:
  - port: 3306
    targetPort: 3306
```
save the file using `ESCAPE + :wq!`

Apply:

```
kubectl apply -f mariadb-svc.yaml
```

---

### Step 4 — Deploy WordPress
Note: Deploy the WordPress frontend configured to connect to the `mariadb` Service.

```
vi wordpress.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: wordpress-lab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        env:
        - name: WORDPRESS_DB_HOST
          value: mariadb
        - name: WORDPRESS_DB_USER
          value: wpuser
        - name: WORDPRESS_DB_PASSWORD
          value: wppass
        - name: WORDPRESS_DB_NAME
          value: wordpress
        ports:
        - containerPort: 80
```
save the file using `ESCAPE + :wq!`

Apply:

```
kubectl apply -f wordpress.yaml
```

---

### Step 5 — Create WordPress Service
Note: Make WordPress reachable (in this lab via `NodePort`) so you can inspect logs and behavior.

```
wordpress-svc.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: wordpress-lab
spec:
  selector:
    app: wordpress
  ports:
  - port: 80
    targetPort: 80
  type: NodePort
```
save the file using `ESCAPE + :wq!`

Apply:

```
kubectl apply -f wordpress-svc.yaml
```

---

### Step 6 — Verify Pods
Note: Confirm both application and database pods are running before testing connectivity.

Command:

```
kubectl get pods -n wordpress-lab
```

Expected: `wordpress-...` and `mariadb-...` should be in `Running` state.

---

### Step 7 — Verify WordPress → MariaDB connectivity

Note: Sanity-check the WordPress logs to ensure no DB connection errors and list services.

Commands:

```
kubectl logs deploy/wordpress -n wordpress-lab
kubectl get svc -n wordpress-lab
```

What to look for: absence of database connection errors in logs and a `mariadb` ClusterIP service.

#### Browser access (NodePort):

1. Get the NodePort assigned to the `wordpress` Service.
2. Find a reachable node IP (ExternalIP or PublicIP).
3. Use the URL and open it in your browser:

```
http://<NodeIP>:<NodePort>
```

Example:

```
http://100.92.0.40:31058
```

Note: Use the Node IP and NodePort to access the WordPress frontend in a browser for manual verification.

---


### Step 8 — Create an "attacker" pod for testing
Note: Simulate an arbitrary pod trying to reach the DB to demonstrate policy enforcement.

Create the pod:

```
kubectl run attacker --image=busybox -n wordpress-lab -- sleep 3600
```

Exec into attacker:

```
kubectl exec -it attacker -n wordpress-lab -- sh
```

Test connectivity (inside attacker):

```
nc -zv mariadb 3306
```

Expected (before NetworkPolicy): connection succeeds.

---

### Step 9 — Apply NetworkPolicy to allow only WordPress → MariaDB
Note: Restrict ingress to MariaDB pods so only pods labeled `app=wordpress` may connect on TCP/3306.

```
vi allow-wordpress-db.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-wordpress-to-db
  namespace: wordpress-lab
spec:
  podSelector:
    matchLabels:
      app: mariadb
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: wordpress
    ports:
    - protocol: TCP
      port: 3306
```
save the file using `ESCAPE + :wq!`

Apply:

```
kubectl apply -f allow-wordpress-db.yaml
```

---

### Step 10 — Verify enforcement
Note: Confirm WordPress can still reach MariaDB while the attacker pod cannot.

WordPress test (from WordPress pod):

1. Get WordPress pod name:

```
kubectl get pod -n wordpress-lab -l app=wordpress
```

2. Exec into WordPress pod (example):

```
kubectl exec -it <wordpress-pod> -n wordpress-lab -- bash
```

3. Install/use a TCP probe then test:

```
apt update && apt install -y netcat-openbsd
nc -zv mariadb 3306
```

Expected: `Connected` (WordPress → MariaDB allowed).

Attacker test (from `attacker` pod):

```
kubectl exec -it attacker -n wordpress-lab -- sh
nc -zv mariadb 3306
```

Expected: `connection timed out` or similar error (attacker → MariaDB denied).

---

## Verify NetworkPolicy object
Note: Inspect the NetworkPolicy to review targeted pods, allowed sources, and ports.

Command:

```
kubectl describe networkpolicy allow-wordpress-to-db -n wordpress-lab
```

What to see:

-+- Target Pod: `app=mariadb`
- Allowed From: `app=wordpress`
- Port: TCP 3306

---

## Production hardening (recommendation)
Note: Always pair allow rules with an explicit default deny to close unintended ingress.

Add a default deny:

Add the given content, by pressing `INSERT`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: wordpress-lab
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

Keep the `allow-wordpress-to-db` policy in place to allow required traffic.

---

## Troubleshooting tips

- If the `attacker` pod can still reach the DB after applying the policy, verify your CNI plugin supports NetworkPolicy. Some bare-bones plugins do not.
- Use `kubectl get pods --show-labels -n wordpress-lab` to confirm labels match the selectors used in the policy.
- Confirm there are no other Services / external endpoints bypassing the ClusterIP.

---

## Cleanup

Remove the lab namespace to delete all resources created for this exercise:

```
kubectl delete ns wordpress-lab
```

---

## Summary

This lab shows how to implement a minimal Zero Trust segmentation using Kubernetes NetworkPolicy: only the application tier (`app=wordpress`) may reach the database tier (`app=mariadb`) on TCP/3306; all other pods are denied.

If you want, I can also add a short section showing how to test this with `kubectl port-forward` or a Kubernetes job-based probe. Would you like that?
