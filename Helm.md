# 🪄 Helm 3 & WordPress Deployment on Kubernetes - LAB Guide

> 💡 **Objective:**  
> In this lab, you will learn how to **install Helm 3**, configure it with your Kubernetes cluster, and deploy a **WordPress application** using the **Bitnami Helm chart**.

---

## 📘 What is Helm?

**Helm** is the **package manager for Kubernetes**, similar to how `apt` or `yum` work for Linux.  
It allows you to define, install, and upgrade even the most complex Kubernetes applications using **Helm charts** — preconfigured YAML templates.

### ⚙️ Helm Concepts

| Term | Description |
|------|--------------|
| **Chart** | A Helm package containing Kubernetes manifests |
| **Repository** | A collection of Helm charts (like Bitnami) |
| **Release** | A running instance of a chart in a Kubernetes cluster |

📦 **In short:** Helm = Kubernetes + Simplicity 🎯

---

## ### 🧩 Task 1: Installing Helm 3 on Kubernetes

### 🔹 Step 1: Update System Packages
```bash
apt update
```
🔹 Step 2: Download Helm Binary

Download the Helm binary for Linux 386 architecture:
```bash
wget https://get.helm.sh/helm-v3.11.3-linux-386.tar.gz
```
🔹 Step 3: Extract the Tarball
```bash
tar -xvzf helm-v3.11.3-linux-386.tar.gz
```
🔹 Step 4: Verify Extraction

Check the extracted files to ensure Helm binary exists:
```bash
ls
```
🔹 Step 5: Move Helm Binary to /bin

Make Helm accessible system-wide:
```bash
mv linux-386/helm /bin/
```

🔹 Step 6: Verify Installation

Confirm Helm is installed correctly:
```bash
helm version
```

✅ Output Example:

version.BuildInfo{Version:"v3.11.3", GitCommit:"...", GoVersion:"go1.20"}

### 🌐 Task 2: Install WordPress using Helm
🔹 Step 1: Verify Helm Installation
```bash
helm version
```

🔹 Step 2: Add Bitnami Helm Repository

Add the official Bitnami charts repository:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```
🔹 Step 3: Verify Repositories
```bash
helm repo list
```

🔹 Step 4: Install WordPress Chart

Install the WordPress application with the release name my-wordpress:

```bash
helm install my-wordpress bitnami/wordpress
```
🔹 Step 5: List Helm Releases
```bash
helm list
```
🧾 Verification Steps
🔹 Step 1: Check Kubernetes Resources
```bash
kubectl get all
```
🔹 Step 2: Verify Pods

Ensure all pods are running:

```bash
kubectl get pods
```

🔹 Step 3: Verify Deployment

Check if the frontend WordPress pods are part of a deployment:
```bash
kubectl get deploy
```

🔹 Step 4: Verify Services
```bash
kubectl get svc
```

If your cluster supports LoadBalancer, note the EXTERNAL-IP to access WordPress.

⚙️ Modify Service Type (Optional)

Change the WordPress service type from LoadBalancer to NodePort:
```bash
kubectl edit svc my-wordpress
```

Then verify the changes:
```bash
kubectl get svc
```

🌍 Access WordPress:

```bash
http://<Public-IP-of-Node>:<NodePort>
```
> e.g. http://3.85.9.5:32123

### 🧹 Cleanup

🔹 Step 1: List Installed Helm Releases
```bash
helm ls
```
🔹 Step 2: Uninstall WordPress

Remove the release:
```bash
helm uninstall my-wordpress
```

🔹 Step 3: Confirm Deletion
```bash
helm ls
```
🔹 Step 4: Remove Repository
```bash
helm repo remove bitnami
```

🔹 Step 5: Remove Helm (Optional)
```bash
sudo apt-get remove helm
```
 
