# 🚀 Kubernetes + CRI-O Installation Lab Guide

This guide walks you through setting up a **Kubernetes cluster (v1.32)** with **CRI-O (v1.32)** as the container runtime on Ubuntu-based systems using `deb` packages.
## 🧱 Environment Setup

> **Note:** Run all commands as `root` or with `sudo`.

### Variables

```bash
KUBERNETES_VERSION=v1.32
CRIO_VERSION=v1.32
```

### ⚙️Install Dependencies for adding repositories

```bash
apt-get update
```
```bash
apt-get install -y software-properties-common curl
```

### 📦 Add Kubernetes Repository
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list
```

### 🐧 Step 3: Add CRI-O Repository
```bash
curl -fsSL https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
```
```bash
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/cri-o.list
```

### 🧩 Install the packages
```bash
apt-get update
```
```bash
apt-get install -y cri-o kubelet kubeadm kubectl
```
### 🔥 Start and Verify CRI-O
```bash
systemctl start crio.service
```
### Check installed versions:

```bash
kubeadm version
```
```bash
kubectl version
```
```bash
kubelet --version
```
```bash
crio version
```

### 🚫 Disable Swap & Configure Networking

```bash
swapoff -a
modprobe br_netfilter
sysctl -w net.ipv4.ip_forward=1
```


### 🧭 Initialize Master Node
> Run this on the **master node only**

```bash
kubeadm init --ignore-preflight-errors=all
```
> If the initialization completes successfully, it will display a **join command** that can be used by worker nodes to connect to the master.
Make sure to **note down the highlighted join command**.
Next, run the following commands to configure `kubectl` on the master node.

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
chown $(id -u):$(id -g) $HOME/.kube/config
```
### 🧩 Joining a Cluster

On each **worker node**, run the `kubeadm join` command that was displayed on the master node during the `kubeadm init` process.

```bash
kubeadm join --token <your_token> --discovery-token-ca-cert-hash <your_discovery_token>
```
> i.e kubeadm join 172.31.28.41:6443 --token ckj3o6.afub3lnqg62ulqek  --discovery-token-ca-cert-hash sha256:ad9cf556679b25609cba17eef5a7cc45fed04b2c2be9f776e33ef03876c4da36 --ignore-preflight-errors=all

**💡 Note:** If you need to regenerate the join token, run the following commands on the master node:

```bash
kubeadm token create --print-join-command
```
Once the worker nodes have successfully joined, verify the node status on the **master node:**

```bash
kubectl get nodes
```
### 🌍 Deploy Container Networking Interface (CNI)

Now, apply the Weave Net CNI plugin from the master node to enable pod-to-pod communication.

```bash
kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.29/net.yaml
```

**📘 Reference:**
[Weaveworks CNI Documentation](https://github.com/rajch/weave#using-weave-on-kubernetes)

**✅ Verify the Setup**

Check the node status — all nodes should be in the **Ready** state:
```bash
kubectl get nodes
```

Check that all system pods (including DNS and Weave) are running:
```bash
kubectl get pods -n kube-system
```

### 🐳 Create and Test the Pod

Create a pod named httpd using the Apache HTTP Server image:

```bash
kubectl run httpd --image=httpd
```

Ensure the pod is in a Running state:
```bash
kubectl get pods
```

Access the Pod Container

Open an interactive shell session inside the container:

```bash
kubectl exec -it <pod_name> -- /bin/bash
```
Install and Test Curl Inside the Container

```bash
apt update
```
```bash
apt install curl -y
```
```bash
curl localhost
```
```bash
exit
```
You should see a response from the Apache HTTP Server, confirming successful deployment.

### ⚠️ Troubleshooting

If you encounter the following error while running `kubectl` commands:
```pgsql
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

Run the command below to configure the correct Kubernetes context:
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

### ⚙️ Cluster Reset (Use with Caution)

> ⚠️ Warning:
> The following command will remove all nodes and reset your cluster configuration.
> Execute only if you are certain.

```bash
kubeadm reset
```

---

### 📚 Additional Reference

If you’d like to explore the **latest official steps** and updates for setting up Kubernetes with **CRI-O**,  
please visit the official CRI-O website:  

🔗 **[https://cri-o.io/](https://cri-o.io/)**

---

 
