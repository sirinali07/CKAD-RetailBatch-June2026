# 🚀 Bootstrap a Kubernetes Cluster using Kubeadm & CRI-O

This lab will guide you through deploying a **Kubernetes cluster (v1.32)** using **CRI-O (v1.32)** as the container runtime on AWS EC2 instances.

---

## 🧱 Task 1: Launch Instances on AWS

Login to your **AWS Console** and create the following instances:

| Count | Instance Type | OS Version          | Storage | Description |
|--------|----------------|---------------------|----------|--------------|
| 3      | t2.medium       | Ubuntu 22.04 LTS    | 10 GB    | 1 Master + 2 Worker Nodes |

### 🔒 Security Group Configuration

Instead of opening all ports, only open the following **Custom TCP** ports:

|      Nodes	      |    Port Number	 |         Use Case                       |
|---------------------|------------------|----------------------------------------|
| Master, Workers	  |    `2379-2380`   |  Etcd Client API / Server API          |
| Master              |       `6443`  	 |  Kubernetes API Server (Secure Port)   |
| Master, Workers     |   `6782-6784`    |  Weave Net Server/Client API #CNI      |
| Master, Workers     |   `10248-10259`	 |  Kubelet Communication                 |
| Workers             |   `30000-32767`	 |  Reserved of NodePort IPs              |	   



## ⚙️ Task 2: Setting up Machines

All steps in this task must be performed on **all the nodes**.

### 🔗 Connect to Instances

Use **PuTTY** (or any SSH client) to connect to each instance and switch to root:

```bash
sudo su
```

#### 🏷️ Rename the VM's as Master, Node1 and Node2 

Set the hostname to all three nodes as master, Node1, and Node2 in their respective terminals for easy understanding, by running the below command:

**Connect to Master.**

Switch to root.
```
sudo su
```
```
hostnamectl set-hostname Master
```
```
bash
```

**Connect to Node1.**

Switch to root.
```
sudo su
```
```
hostnamectl set-hostname Node1
```
```
bash
```

**Connect to Node2.**

Switch to root.
```
sudo su
```
```
hostnamectl set-hostname Node2
```
```
bash
```

#### 🧩 Install the core Kubernetes components and container runtime
> This step installs:
> - **kubeadm :** for cluster bootstrapping
> - **kubelet :** the node agent that runs pods
> - **kubectl :** the command-line tool to manage the cluster
> - **cri-o   :** lightweight container runtime optimized for Kubernetes

**Variables**

```bash
KUBERNETES_VERSION=v1.32
CRIO_VERSION=v1.32
```

**⚙️Install Dependencies for adding repositories**

```bash
apt-get update
```
```bash
apt-get install -y software-properties-common curl
```

**📦 Add Kubernetes Repository**
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list
```

**🐧 Add CRI-O Repository**
```bash
curl -fsSL https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
```
```bash
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/cri-o.list
```

**🧩 Install the packages**
```bash
apt-get update
```
```bash
apt-get install -y cri-o kubelet kubeadm kubectl
```
**🔥 Start and Verify CRI-O**
```bash
systemctl start crio.service
```
**Check installed versions:**

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

**🚫 Disable Swap & Configure Networking**

```bash
swapoff -a
modprobe br_netfilter
sysctl -w net.ipv4.ip_forward=1
```



### Task 3: Initializing the Cluster

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

### Task 4: Joining a Cluster

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

### Task 5: Deploy Container Networking Interface

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

### Task 6: Create Pods

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

## ⚠️ Troubleshooting

If you encounter the following error while running `kubectl` commands:
```pgsql
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

Run the command below to configure the correct Kubernetes context:
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## ⚙️ Cluster Reset (Use with Caution)

> ⚠️ Warning:
> The following command will remove all nodes and reset your cluster configuration.
> Execute only if you are certain.

```bash
kubeadm reset
```

---

## 📚 Additional Reference

If you’d like to explore the **latest official steps** and updates for setting up Kubernetes with **CRI-O**,  
please visit the official CRI-O website:  

🔗 **[https://cri-o.io/](https://cri-o.io/)**

---

**End of Lab — Happy Learning! 🚀**

