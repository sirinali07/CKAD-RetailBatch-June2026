## ConfigMap & Secrets
In Kubernetes, applications should be **configuration-driven,** not hard-coded.
**ConfigMaps** and **Secrets** help achieve this by separating configuration data from application code.

**ConfigMap**
A *ConfigMap* is used to store non-sensitive configuration data such as:
- Database usernames
- Application settings
- Feature flags
- Environment-specific values

**Secret**
A *Secret* stores sensitive information(Secrets are stored in *base64-encoded* format), such as:
- Passwords
- API tokens
- Certificates
- Database credentials

----------------------------------------------------------------

### Task 1: Directly inject variables - Traditional Method
```
vi env.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: ws
  name: env-pod
spec:
  containers:
  - image: nginx
    name: ng-ctr
    ports:
    - containerPort: 80
    env:
    - name: db_user   #key
      value: admin
    - name: db_pwd
      value: "1234"
```
save the file using `ESCAPE + :wq!`

Apply the YAML file
```
kubectl apply -f env.yaml
```
```
kubectl describe pod env-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it env-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
env | grep db_
```

### Task 2: Inject `ALL` variables from ConfigMaps(FromLiteral) into POD.
Create a ConfigMap
```
kubectl create cm cm-1 --from-literal=db_user=admin --from-literal=db_pwd=1234
```
> Each `--from-literal` creates a key-value pair inside the ConfigMap.

Verify ConfigMap
```
kubectl get cm
```
```
kubectl describe cm cm-1
```
Inject the ConfigMap into the Pod Yaml File
```
vi env.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: cm-1
```
save the file using `ESCAPE + :wq!`

> `envFrom` injects `all keys` from the ConfigMap as environment variables.

Apply the YAML file
```
kubectl apply -f env.yaml
```

```
kubectl describe pod web-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
env | grep db_
```



### Task 3 : Injecting ConfigMap as volume mount

Create a file
```
vi token
```
```
This is CKAD Training. We are practicing Injecting variables from ConfigMaps(FromFile) into POD.
```
Create a ConfigMap
```
kubectl create cm file-cm --from-file=token        
```
```
kubectl get cm
```
```
kubectl describe cm file-cm 
```
Inject as volume mount
```
vi env.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  volumes:
  - name: cm-volume
    configMap:
      name: file-cm 
  containers:
  - image: httpd
    name: ctr-1
    volumeMounts:
    - name: cm-volume
      mountPath: /app
    ports:
    - containerPort: 80

```
save the file using `ESCAPE + :wq!`

Apply the YAML file

```
kubectl apply -f env.yaml
```
```
kubectl describe pod web-pod
```
Verify Mounted File
```
kubectl exec -it web-pod -- sh
```
```
cd /app
```
```
cat token
```
> ConfigMap key appears as a file inside the container.
```
exit
```

### Task 4 : Secret
Imperative
```
kubectl create secret generic secret-1 --from-literal=db_user=admin --from-literal=db_pwd=123
```
> Secrets are base64-encoded, not encrypted by default.
> Each `--from-literal` creates a key-value pair inside the Secret.

```
kubectl get secret
```
```
kubectl describe secret secret-1
```
Declrative
```
vi secret.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
data:
  ##all below values are base64 encoded

  ## To encode and print
  ## echo -n '<value-to-be-encoded>' | base64

  ## To decode and print
  ## echo '<value-to-be-decoded>' | base64 -d

  ##rootpw is root
  rootpw: cm9vdAo=

  ##user is user
  user: dXNlcgo=

  ##password is mypwd
  password: bXlwd2QK
```
save the file using `ESCAPE + :wq!`

Apply the YAML file

```
kubectl apply -f secret.yaml
```
```
kubectl get secrets
```
```
kubectl describe secrets mysql-credentials
```
You can inject the secret in all the three ways as above.

Injecting all values
```
vi sc-pod.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sc-pod
  name: sc-pod
spec:
  containers:
  - image: nginx
    name: sc-ctr
    envFrom:
    - secretRef:
        name: secret-1
    - secretRef:
        name: mysql-credentials
```
save the file using `ESCAPE + :wq!`

Apply the YAML file

```
kubectl apply -f sc-pod.yaml
```
```
kubectl get po
```
```
kubectl exec -it sc-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
echo $rootpw
```
```
echo $user
```
```
echo $password
```
```
env 
```
```
exit
```
