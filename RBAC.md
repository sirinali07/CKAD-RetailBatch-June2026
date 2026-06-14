## RBAC

RBAC stands for Role-Based Access Control. 

It is a method of regulating access to resources based on the roles of individual users within an organization. With RBAC, you can specify what actions (such as create, read, update, delete) a user or group of users can perform on specific Kubernetes resources.

**Roles:** A role is a set of permissions that define what actions can be performed on Kubernetes resources within a namespace. For example, you can define a role that allows read-only access to pods or a role that allows full control over deployments.

**RoleBindings:** A role binding binds a role to a user, group, or service account within a namespace. It specifies who has access to the resources defined by the role. For example, you can create a role binding that associates a specific role with a particular user, granting them the permissions defined by that role.

**Cluster Role:** A ClusterRole is a set of permissions that can be applied across the entire Kubernetes cluster.ClusterRoles are not namespace-scoped; they apply cluster-wide. This means that permissions granted by a ClusterRole affect resources in all namespaces.

**Cluster Role Binding:** A ClusterRoleBinding binds a ClusterRole to a set of subjects (users, groups, or service accounts) across the entire cluster. It effectively grants the permissions defined in the ClusterRole to the subjects specified in the binding. Similar to ClusterRoles, ClusterRoleBindings are also cluster-wide and apply across all namespaces.


## Task 1: Launching the Kubernetes Dashboard using Cluster Role Binding

The Kubernetes Dashboard is a web-based user interface for Kubernetes clusters. It provides a graphical interface for users to manage and monitor their Kubernetes clusters and applications running within them.

The dashboard user interface is not deployed by default. To deploy it, run the following command:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```
Enter the following commands to verify if the pods, services, and deployments have been created:
```
kubectl get pods -n kubernetes-dashboard -o wide
```
```
kubectl get deployment -n kubernetes-dashboard -o wide
```
```
kubectl get svc -n kubernetes-dashboard -o wide
```
To access the service outside the cluster, edit the **service type** from `ClusterIP to LoadBalancer` using the following command:
```
kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard
```
To confirm that the service **type** has been changed to **LoadBalancer**
```
kubectl get svc -n kubernetes-dashboard -o wide
```
Access Kubernetes Dashboard

Open browser and access:
> https://<EXTERNAL-IP>
⚠️ Browser Security:
- Click Advanced
- Click Accept the Risk and Continue
  
## Task 2: Create ServiceAccount for Dashboard Access

Create a service account by running the following command, and then input the code in the master node:
```
vi ServiceAccount.yaml
```
Add the given content, by pressing `INSERT`
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```
save the file using `ESCAPE + :wq!`

Apply the YAML file with the command:
```
kubectl apply -f ServiceAccount.yaml
```
ServiceAccount represents a machine identity used by Dashboard.

## Task 3: Create ClusterRole and  ClusterRole Binding
Create ClusterRole YAML
```
vi clusterrole.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: full-access-role
rules:
- apiGroups: ["*"]          # All API groups
  resources: ["*"]          # All resources
  verbs: ["*"]
```
save the file using `ESCAPE + :wq!`

Apply ClusterRole YAML
```
kubectl apply -f clusterrole.yaml
```
Verify ClusterRole
```
kubectl get clusterrole full-access-role
```
Create a yaml file for cluster role binding using below command and code:
```
vi ClusterRoleBinding.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cr-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: full-access-role      # Custom full privilege role
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
save the file using `ESCAPE + :wq!`

Run the below command to create cluster role binding:
```
kubectl apply -f ClusterRoleBinding.yaml
```
## Task 4: Generate Login Token
Create the token by running the following command:

```
kubectl -n kubernetes-dashboard create token admin-user
```
Copy the token and paste it into the **Kubernetes dashboard** login screen, then click **Sign in**

## Task 5: Cleanup
To delete the Kubernetes dashboard version 2.5, use the following command in the master node:
```
kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
```
By following these steps, you will be able to deploy the Kubernetes Dashboard, establish secure access, and navigate the interface to monitor and manage your Kubernetes cluster.


