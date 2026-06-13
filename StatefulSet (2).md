## StatefulSet

A **StatefulSe**t is similar to a Deployment but is designed for applications where ***each pod must maintain a unique identity and its own persistent data,*** even if the pod is deleted or restarted.

**It is mainly used for:**
* Databases (MySQL, MongoDB, PostgreSQL)
* Distributed systems (Kafka, Zookeeper, Cassandra, Redis)
* Any workload that needs stable storage + predictable pod names

<img width="945" height="268" alt="image" src="https://github.com/user-attachments/assets/47a2b659-d725-435e-9ed6-9e8478ab3474" />

**Key Features of StatefulSet:**
* Stable, predictable pod names (pod-0, pod-1, pod-2).
* Ordered deployment (pods start sequentially: 0 → 1 → 2).
* Ordered termination (pods stop in reverse order: 2 → 1 → 0).
* Stable network identity for each pod via DNS.
* Requires a headless service (clusterIP: None) for pod DNS resolution.
* Persistent storage per pod using volumeClaimTemplates.
* PVCs remain even after pods are deleted (data is preserved).
* Each pod gets its own dedicated volume (no sharing unless explicitly allowed).
* Rolling updates happen one pod at a time for stability.
Best suited for stateful applications like databases and distributed systems.

---

### Task 1: Create Stateful Set
Create the yaml definition for an nginx Stateful Set 
```
vi nginx-sts.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx-svc
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx-sts
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-sts
spec:
  serviceName: nginx-svc
  replicas: 2
  selector:
    matchLabels:
      app: nginx-sts
  template:
    metadata:
      labels:
        app: nginx-sts
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
Create a Stateful Set and  headless service by applying the yaml
```
kubectl apply -f nginx-sts.yaml
```
Validate the headless service creation
```
kubectl get service nginx-svc
```
Validate the stateful set creation
```
kubectl get statefulset nginx-sts
```
Watch the pods getting created in an ordinal index fashion
```
kubectl get pods -w -l app=nginx-sts
```
Create a busybox pod to test the ping to one of the Ngninx pods by using DNS name of the pod
```
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
```
```
nslookup nginx-sts-0.nginx-svc
```
```
nslookup nginx-sts-1.nginx-svc
```
```
exit
```
Delete the pods for a stateful set
```
kubectl delete pod -l app=nginx-sts
```
In another window notice the new pods getting created in a proper order
```
kubectl get pod -w -l app=nginx-sts
```

### Task 2: Scaling a Stateful Set
Scale the Stateful Set to 5 replicas using below.
```
kubectl scale sts nginx-sts --replicas=5
```
Verify the pods getting created in ordinal way
```
kubectl get pods -w -l app=nginx-sts
```
Verify the PV Claim getting created in ordinal fashion
```
kubectl get pvc -l app=nginx-sts
```
Edit the stateful Set yam and reduce replicas to 3 
```
kubectl edit sts nginx-sts
```
Notice that the controller deletes the pods one at a time. It waits for one to completely shut down before going to next
```
kubectl get pods -w -l app=nginx-sts
```
Verify statefulSet’s PersistentVolumeClaims and verify that are not deleted on scaling down. 
```
kubectl get pvc -l app=nginx-sts
```

### Task 3: Cleanup the resources using below command 
Delete a Stateful Set
```
kubectl delete -f nginx-sts.yaml
```
List all the PV and PVC’s that has been allocated to Statefulset pods and delete them as below.
```
kubectl get pvc
```
```
kubectl delete pvc --all
```



