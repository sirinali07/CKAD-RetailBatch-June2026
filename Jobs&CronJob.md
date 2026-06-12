## Jobs and Cronjobs in Kubernetes

### Jobs in Kubernetes

A **Job** in Kubernetes is used to run a task to completion. Unlike regular Pods (which may run forever), a Job ensures that a specific task runs successfully and then stops.

**Key Characteristics:**
* Runs a Pod until the task completes
* Ensures successful completion (retries if needed)
* Creates one or more Pods to complete the task
* Stops once the task is done

**Use Cases:**
* Batch processing
* Data migration tasks
* One-time scripts (like database setup)
* Sending emails or reports

**How it Works:**
* You define a Job with a Pod template
* Kubernetes creates a Pod
* If the Pod fails, Kubernetes retries automatically
* When the task succeeds, the Job is marked Completed

---

### CronJobs in Kubernetes

A **CronJob** is used to run Jobs on a schedule, similar to cron jobs in Linux.

It uses the same concept as the Unix **cron scheduler.

**Key Characteristics:**
* Runs Jobs periodically
* Uses cron expressions for scheduling
* Automatically creates Jobs at specified times

**Use Cases:**
* Daily backups
* Log rotation
* Sending periodic notifications
* Cleanup tasks

**How it Works:**
* You define a schedule (e.g., every minute)
* Kubernetes creates a Job at each scheduled time
* Each Job creates a Pod to perform the task

**🕒 Cron Schedule Format**

<img width="596" height="306" alt="image" src="https://github.com/user-attachments/assets/e51bca02-b547-411b-ab36-e4a7b923600a" />


**Example:**

> `*/1 * * * *` → Runs **every minute**

<img width="734" height="217" alt="Screenshot 2026-04-01 at 9 57 02 AM" src="https://github.com/user-attachments/assets/add9a40c-e744-4e21-8e5a-cacb9a38326b" />

---

### Task 1: Jobs in Kubernetes 

Create a `job-pod.yaml` using content given below
```
vi job-pod.yaml
```
Add the given content, by pressing `INSERT`
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: jobs-hello
spec:
  template:
    metadata:
      name: jobs-pod
    spec:
      containers:
      - name: jobs-ctr
        image: busybox
        args:
        - /bin/sh
        - -c
        - "echo HELLO WORLD !!!!!"
      restartPolicy: Never
```
Apply the  YAML
```
kubectl apply -f job-pod.yaml
```
```
kubectl get jobs
```
```
kubectl get pods
```

Read logs 
```
kubectl logs <name of jobpod>
```

Describe the job
```
kubectl describe jobs jobs-hello
```
```
kubectl describe pod <pod-name>
```
Delete the job
```
kubectl delete -f job-pod.yaml
```

### Task 2: Cronjobs 

Create a yaml called `cronjob.yaml`. Use the content given below to fill the file
```
vi cronjob.yaml
```
Add the given content, by pressing `INSERT`
```yaml
 
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cronjob-ctr
            image: busybox
            args:
            - /bin/sh
            - -c
            - "echo Hello World!"
          restartPolicy: OnFailure
```
save the file using `ESCAPE + :wq!`

```
kubectl get po
```
```
kubectl get job
```
```
kubectl get cronjobs 
```
Apply the  YAML
```
kubectl apply -f cronjob.yaml
```
```
kubectl get cronjobs.batch
```
```
kubectl get pod
```
```
kubectl logs <cronjob-pod-name>
```
In new tabs , you can add a watch to job and pods by the below command
```
kubectl get po -w
```
```
kubectl get jobs -w
```
To check the number of executions of the cronjobs
```
kubectl describe cronjobs
```
To edit the running cronjob
```
kubectl edit cronjob <cronjob-name>
```
To generate CronJob schedule expressions, you can also use web tools like https://crontab.guru/
