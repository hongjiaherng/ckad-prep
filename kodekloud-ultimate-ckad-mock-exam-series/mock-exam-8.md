# Lab: CKAD Mock Exam 8

## Application Design And Build

### Question 1

**Context:** `kubectl config use-context cluster1`

**Task:**
There is a requirement to share a volume between two containers that are running within the same pod. Use the following instructions to create the pod and related objects:

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create required Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: grape-pod-ckad06-str
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: grape-vol-ckad06-str
      mountPath: /var/log/nginx
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "sleep 10000"]
    volumeMounts:
    - name: grape-vol-ckad06-str
      mountPath: /usr/src
  volumes:
  - name: grape-vol-ckad06-str
    emptyDir: {}
```

</details>

**Details (Verification):**

* Do the main container of the pod use correct image?
* Does the main container uses correct mount?
* Does the co-located container uses correct mount?
* Is the volume created correctly?
* Is the pod is running?

-----

### Question 2

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-job` namespace, schedule a job called `learning-every-minute` that prints this message in the shell every minute: `I am practicing for CKAD certification`.

In case the container in pod failed for any reason, it should be restarted automatically.

Use `busybox:1.28` image for the cronjob!

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  namespace: ckad-job
  name: learning-every-minute
spec:
  schedule: "*/1 * * * *"  # Runs every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: learning-every-minute
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - echo I am practicing for CKAD certification
          restartPolicy: OnFailure
```

Then use `kubectl apply -f file_name.yaml` to create the required object.

</details>

**Details (Verification):**

* Is the cronjob `learning-every-minute` created?
* Is the container image `busybox:1.28`?
* Does cronjob run required command?
* Does cronjob run every minute?

-----

### Question 3

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-pod-design` namespace, we created a pod named `basic-nginx` that runs the `nginx:1.17` image.
  
Take appropriate actions to update the `index.html` page of this NGINX container with below value instead of default NGINX welcome page:

```bash
Hello from KodeKloud!
```

<details>
<summary><b>View Solution</b></summary>

Exec to the pod container and update the index.html file content:

```bash
student-node ~ ➜kubectl exec -it -n ckad-pod-design basic-nginx -- sh
# echo 'Hello from KodeKloud!' > /usr/share/nginx/html/index.html
```

Observe the result:

```bash
student-node ~ ➜  kubectl exec -it -n ckad-pod-design basic-nginx -- cat /usr/share/nginx/html/index.html
Hello from KodeKloud!
```

</details>

**Details (Verification):**

* Is pod `basic-nginx` running?
* Is the `index.html` file updated properly?

-----

### Question 4

**Context:** `kubectl config use-context cluster1`

**Task:**
A persistent volume called `app-data-pv` is already created with a storage capacity of `60Mi`. It's using the `high-performance-sc` storage class with the path `/opt/high-performance-sc`.  
  
Also, a persistent volume claim named `app-data-pvc` has been created on this cluster. This PVC has requested `60Mi` of storage from `app-data-pv` volume.  
  
Resize the PVC to `100Mi` and make sure the PVC is in `Bound` state.

<details>
<summary><b>View Solution</b></summary>

Edit `app-data-pv` PV:

```bash
kubectl get pv app-data-pv -o yaml > /tmp/app-data-pv.yaml
```

Edit the template:

```bash
vi /tmp/app-data-pv.yaml
```

Delete all entries for `uid:`, `annotations`, `status:`, `claimRef:` from the template.

Change `storage: 60Mi` to `storage: 100Mi` and save the template.  
  
Edit `app-data-pvc` PVC:

```bash
kubectl get pvc app-data-pvc -o yaml > /tmp/app-data-pvc.yaml
```

Edit the template:

```bash
vi /tmp/app-data-pvc.yaml
```

Delete all entries for `uid:`, `annotations`, `status:` from the template.  
  
Under `resources:` -> `requests:` change `storage: 60Mi` to `storage: 100Mi` and save the template.

Delete the exsiting PVC & PV :

```bash
kubectl delete pvc app-data-pvc
kubectl delete pv app-data-pv
```

Create the PVC & PV using the template:

```bash
kubectl apply -f /tmp/app-data-pv.yaml
kubectl apply -f /tmp/app-data-pvc.yaml
```

</details>

**Details (Verification):**

* Is `app-data-pvc` PVC resized?

-----

### Question 5

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a storage class with the name `database-sc` as per the properties given below:  
  
* Provisioner should be `kubernetes.io/azure-file`,  
  
* reclaimPolicy should be `Retain.  
  
* Volume expansion should be enabled.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create desired storage-class:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: database-sc
provisioner: kubernetes.io/azure-file
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

</details>

**Details (Verification):**

* Is `database-sc` storage class created?

-----

## Application Deployment

### Question 6

**Context:** `kubectl config use-context cluster1`

**Task:**
One application deployment called `deluxe-apd` got deployed on the wrong namespace. The team wants you to take the appropriate steps to deploy it on the `dove-ns-8978f5ff7` namespace.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster1
```

In this task, we will use the `kubectl` command. Here are the steps: -

1. Use the `kubectl get` command to list the deployments on all the namespaces.

```bash
kubectl get deployment -A
```

Here `-A` option stands for `all namespaces`, which displays resources from all namespaces.

Inspect the deployment name `deluxe-apd` from the above command output and identify which namespace it got deployed.

2. Use the `kubectl get` command to retrieves the YAML definition of a deployment named `deluxe-apd` and save it into a file.

```bash
kubectl get deployment deluxe-apd -n app-lox12387 -o yaml > <FILE-NAME>.yaml
```

Here `-n` option stands for `namespace`, which is used to specify the namespace.

Open the file with any text editor such as `vi` or `nano` and make the necessary changes and save it. It should look like this: -

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: deluxe-apd
  name: deluxe-apd
  namespace: dove-ns-8978f5ff7 
spec:
  selector:
    matchLabels:
      app: deluxe-apd
  template:
    metadata:
      labels:
        app: deluxe-apd
    spec:
      containers:
      - image: nginx:1.23
        imagePullPolicy: IfNotPresent
        name: nginx-deluxe-apd
```

3. Before deploying on the given namespace, list the namespaces if it exists or not.

```bash
kubectl get ns
```

If it's not exist, then create the namespace first with the following command: -

```bash
kubectl create ns dove-ns-8978f5ff7
```

4. Now, deploy the resource with the following command:

```bash
kubectl create -f <FILE-NAME>.yaml
```

The above command will deploy the resource in the given namespace.

</details>

**Details (Verification):**

* Is app deployed on the correct ns?

-----

### Question 7

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `dev-apd` namespace, one of the developers has performed a rolling update and upgraded the application to a newer version. But somehow, application pods are not being created.

To regain the working state, `rollback` the application to the previous version.

After rolling the deployment back, on the `controlplane` node, save the image currently in use to the `/root/records/rolling-back-record.txt` file and increase the replica count to `4`.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster1
```

In this task, we will use the `kubectl describe`, `kubectl get`, `kubectl rollout` and `kubectl scale` commands. Here are the steps: -

1. First check the status of the pods: -

```bash
kubectl get pods -n dev-apd
```

One of the pods is in an error state. By using the `kubectl describe` command. We can see that there is an issue with the image.
  
2. We can check the revision history of a deployment by using the `kubectl history` command as follows: -

```bash
kubectl rollout history -n dev-apd deploy webapp-apd
```

3. Inspect the revision in detail as follows: -

```bash
kubectl rollout history -n dev-apd deploy webapp-apd --revision=2
```

4. We found that the image issue happened because of the wrong image tag, and the previous image was correct. As a quick fix, we need to roll back to the previous revision. Use the `kubectl rollout` command: -

```bash
kubectl rollout undo -n dev-apd deploy webapp-apd
```

5. After successful rolling back, inspect the updated image: -

```bash
kubectl describe deploy -n dev-apd webapp-apd | grep -i image
```

6. On the `Controlplane` node, save the image name to the given path `/root/records/rolling-back-record.txt`: -

```bash
ssh cluster1-controlplane

echo "kodekloud/webapp-color" > /root/records/rolling-back-record.txt
```

If the `records` directory is absent, use the `mkdir` command to create this directory.

> NOTE: - To exit from any node, type `exit` on the terminal or press `CTRL + D`.

7. And increase the replica count to the `4` with help of `kubectl scale` command: -

```bash
kubectl scale deploy -n dev-apd webapp-apd --replicas=4
```

Verify it by running the command: `kubectl get deploy -n dev-apd`

</details>

**Details (Verification):**

* Is rolling back successful?
* Is the Image saved to a file?
* Is the deployment scaled?

-----

### Question 8

**Context:** `kubectl config use-context cluster1`

**Task:**
On cluster1, a new deployment called `cube-alpha-apd` has been created in the `alpha-ns-apd` namespace using the image `kodekloud/webapp-color:v2`. This deployment will test a newer version of the `alpha` app.

Configure the deployment in such a way that the `alpha-apd-service` service routes less than `40%` of traffic to the new deployment.
  
 **NOTE: -**  Do not increase the replicas of the `ruby-alpha-apd` deployment.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster1
```

In this task, we will use the `kubectl` command. Here are the steps: -

The `cube-alpha-apd` and `ruby-alpha-apd` deployment has 5-5 replicas. The `alpha-apd-service` service now routes traffic to 10 pods in total (5 replicas on the `ruby-alpha-apd` deployment and 5 replicas from `cube-alpha-apd` deployment).

Use the `kubectl get` command to list the following deployments: -

```bash
kubectl get deploy -n alpha-ns-apd
```

Since the service distributes traffic to all pods equally, in this case, approximately 50% of the traffic will go to `cube-alpha-apd` deployment.

To reduce this below `40%`, scale down the pods on the `cube-alpha-apd` deployment to the minimum to `2`.

```bash
kubectl scale deployment --replicas=2 cube-alpha-apd -n alpha-ns-apd
```

Once this is done, only `~40%` of traffic should go to the `v2` version.

</details>

**Details (Verification):**

* Is deployment configured correctly?

-----

### Question 9

**Context:** `kubectl config use-context cluster1`

**Task:**
On cluster1, in the `marketing-ns` namespace, one of the interns deployed one web application called `monitoring-agent`.

After successfully deploying on the worker node, we start getting alerts about the pod crashing.
  
We want you to inspect the `marketing-ns` namespace and fix those issues.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster1
```

In this task, we will use the `kubectl get`, `kubectl describe`, `kubectl logs` and `kubectl edit` commands. Here are the steps: -

1. To check all the resources in the specific namespaces in the `cluster1`, we would have to run the following command:

```bash
kubectl get all -n marketing-ns
```

It will list all the available resources of the `marketing-ns` namespace.

2. We can see that one of the pods is in an error state. Use the `kubectl describe` command to get detailed information of that pod: -

```bash
kubectl describe -n marketing-ns po <POD-NAME>
```

The output of the `kubectl describe` command includes information about the Pod's status, including its IP address, the containers running in the Pod, and their current state.
It also includes information about the Pod's configuration, such as its labels, annotations, and resource requirements.

3. Now, we will use the `kubectl logs` command to retrieve the logs of a container running in a Pod. Use the following command as follows: -

```bash
kubectl logs -n marketing-ns <POD-NAME> <CONTAINER-NAME>
```

Here, `<POD-NAME>` is the name of the Pod in which the container is running, and `<CONTAINER-NAME>` is the name of the container whose logs we want to retrieve. If the Pod has only one container, we can omit the `<CONTAINER-NAME>` argument.

4. In the logs, we will see that there is a typo in the `sleep` command which needs to be correct. Use the `kubectl edit` command as follows: -

```bash
kubectl edit deploy -n marketing-ns monitoring-agent
```

After fixing the typo, press the `ESC` button and type `:wq`. This command will save the changes to the file and then quit `vi` editor.

Run the `kubectl get` command again to check the pod's status.

```bash
kubectl get pods -n marketing-ns
```

The pod should be running.

</details>

**Details (Verification):**

* Is deployment running?

-----

### Question 10

**Context:** `kubectl config use-context cluster1`

**Task:**
We have deployed two applications called `alpha-color-app` and `beta-color-app` on the `default` namespace using the `kodekloud/webapp-color:v1` and `kodekloud/webapp-color:v2`.
  
We have done all the tests and do not want `alpha-color-app` deployment to receive traffic from the `frontend-service` service anymore. So, route all the traffic to another existing deployment.

Do change the service specifications to route traffic to the `beta-color-app` deployment.

You can test the application from the terminal by running the `curl` command with the following syntax: -

```bash
curl http://cluster1-controlplane:NODE-PORT
<!doctype html>
<title>Hello from Flask</title>
...
  <h2>
    Application Version: v2
  </h2>
```

As shown above, we will get the `Application Version: v2` in the output.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster1
```

In this task, we will use the `kubectl` command. Here are the steps: -

1. Use the `kubectl get` command to list all the given resources: -

```bash
kubectl get deploy,svc -n default
```

2. Identify the created deployment and service names.
3. Now, check the label used in the service `selector` field for the deployment named `beta-color-app`.

```bash
kubectl describe service frontend-service
```

4. Check the labels for the deployment named `beta-color-app` deployment: -

```bash
kubectl get deploy beta-color-app -oyaml
```

5. Update the selector label of the service.

```bash
kubectl edit service frontend-service
```

By default, it will open a `VI` editor. After updating the selector labels. Press `ESC` and type `:wq`. It will save the changes and quit the editor.

</details>

**Details (Verification):**

* Is the application passed the test?
* Is the service updated?

-----

## Services And Networking

### Question 11

**Context:** `kubectl config use-context cluster3`

**Task:**
A new payment service has been introduced. Since it is a sensitive application, it is deployed in its own namespace `critical-space`. Inspect the resources and service created.  
  
You are requested to make the new application available at `/pay`. Create an ingress resource named `ingress-ckad09-svcn` for the payment application to make it available at `/pay`

<details>
<summary><b>View Solution</b></summary>

Switch to Cluster3 using the following:

```bash
kubectl config use-context cluster3
```

Solution manifest file to create a new ingress service to make the application available at `/pay` as follows:

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-ckad09-svcn
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
            name: pay-service
            port:
              number: 8282
```

</details>

**Details (Verification):**

* Is ingress `ingress-ckad09-svcn` created?
* Is ingress configured for service `pay-service`?
* is the correct port configured?

-----

### Question 12

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a Deployment named `ckad15-depl-svcn` with "two replicas" of `nginx` image and expose it using a service named `ckad15-service-svcn`.
  
Please be noted that service needs to be accessed from both inside and outside the cluster (use port `30085`).

<details>
<summary><b>View Solution</b></summary>

* The following manifest can be used to create an deployment `ckad15-depl-svcn` with `nginx` image and `2` replicas.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ckad15-depl-svcn
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

* To access from outside the cluster, we use nodeport type of service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ckad15-service-svcn
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30085
```

</details>

**Details (Verification):**

* Is deployment `ckad15-depl-svcn` created?
* Is service `ckad15-service-svcn` created?
* Is service accessible from outside the cluster?

-----

### Question 13

**Context:** `kubectl config use-context cluster3`

**Task:**
For this scenario, create a deployment named `authentication-deployment` using the image `kodekloud/webapp-color` with `3` replicas.  
  
Expose the `authentication-deployment` with service named `authentication-service` on port `31638` on the nodes of the cluster.

<details>
<summary><b>View Solution</b></summary>

Switch to `cluster3` :

```bash
kubectl config use-context cluster3
```

On student-node, use the command: `kubectl create deployment authentication-deployment --image=kodekloud/webapp-color --replicas=3`  
  
Now we can run the command: `kubectl expose deployment authentication-deployment --type=NodePort --port=8080 --name=authentication-service --dry-run=client -o yaml > authentication-service.yaml` to generate a service definition file.

Now, in generated service definition file add the `nodePort` field with the given port number under the `ports` section and create a service.

</details>

**Details (Verification):**

* Is deployment name `authentication-deployment`?
* Is the Image `kodekloud/webapp-color` used?
* Does deployment has `3` replicas?
* Is service `authentication-service` created?
* Is service is of type "NodePort"?
* Does service has 3 Endpoints?
* Is service configured for port `8080`?
* Does service use NodePort `31638`?

-----

## Application Environment,  Configuration And Security

### Question 14

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad13-ns-sa-aecs` namespace, configure the `ckad13-nginx-pod-aecs` Pod to include a **projected volume** named `vault-token`. Mount the service account token to the container at `/var/run/secrets/tokens`, with an expiration time of `9000` seconds.
  
Additionally, set the intended audience for the token to `vault` and path to `vault-token`.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  k get po -n ckad13-ns-sa-aecs ckad13-nginx-pod-aecs -o yaml > ckad-pro-vol.yaml

student-node ~ ➜  cat ckad-pro-vol.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ckad13-nginx-pod-aecs
  namespace: ckad13-ns-sa-aecs
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
.
.
.
   volumeMounts:                              # Added
    - mountPath: /var/run/secrets/tokens       # Added
      name: vault-token                        # Added
.
.
.
  serviceAccount: ckad13-sa-aecs
  serviceAccountName: ckad13-sa-aecs
  volumes:
  - name: vault-token                   # Added
    projected:                          # Added
      sources:                          # Added
      - serviceAccountToken:            # Added
          path: vault-token             # Added
          expirationSeconds: 9000       # Added
          audience: vault               # Added

student-node ~ ➜  k replace -f ckad-pro-vol.yaml --force 
pod "ckad13-nginx-pod-aecs" deleted
pod/ckad13-nginx-pod-aecs replaced
```

</details>

**Details (Verification):**

* Is a "projected volume" ?
* Is volume configured correctly ?
* Is volume mounted correctly ?

-----

### Question 15

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a Kubernetes Pod named `ckad16-memory-aecs`, with a container named `ckad16-memory-ctr-aecs` running the `polinux/stress` image, and configure it to use the following specifications:

* Command: `stress`
* Arguments: `["--vm", "1", "--vm-bytes", "15M", "--vm-hang", "1"]`
* Requested memory: `10Mi`
* Memory limit: `20Mi`

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ckad16-memory-aecs
spec:
  containers:
  - name: ckad16-memory-ctr-aecs
    image: polinux/stress
    resources:
      requests:
        memory: "10Mi"
      limits:
        memory: "20Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "15M", "--vm-hang", "1"]
EOF

pod/ckad16-memory-aecs created
```

</details>

**Details (Verification):**

* Is correct container name and image used?
* Is correct command used ?
* Is correct argument used ?
* Is "request" and "limit" for memory configured ?

-----

### Question 16

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a **ResourceQuota** called `ckad19-rqc-aecs` in the namespace `ckad19-rqc-ns-aecs` and enforce a limit of `one` **ResourceQuota** for the namespace.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl create namespace ckad19-rqc-ns-aecs
namespace/ckad19-rqc-ns-aecs created

student-node ~ ➜  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ckad19-rqc-aecs
  namespace: ckad19-rqc-ns-aecs
spec:
  hard:
    resourcequotas: "1"
EOF

resourcequota/ckad19-rqc-aecs created

student-node ~ ➜  k get resourcequotas -n ckad19-rqc-ns-aecs
NAME              AGE   REQUEST               LIMIT
ckad19-rqc-aecs   20s   resourcequotas: 1/1
```

</details>

**Details (Verification):**

* Is resource quota created?

-----

### Question 17

**Context:** `kubectl config use-context cluster1`

**Task:**
Update the `web` container for the co-located containers pod `ckad05-multi-pod-aecs` in the namespace `ckad05-securityctx-aecs` so that all the processes run with user ID `1002` for **web** container.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  k get pods -n ckad05-securityctx-aecs 
NAME                    READY   STATUS    RESTARTS   AGE
ckad05-multi-pod-aecs   2/2     Running   0          2m47s

student-node ~ ➜  k exec -i -n ckad05-securityctx-aecs ckad05-multi-pod-aecs -c web -- id
uid=1001 gid=0(root) groups=0(root)

student-node ~ ➜  k get pods -n ckad05-securityctx-aecs ckad05-multi-pod-aecs -o yaml > security-ctx-web.yaml

student-node ~ ➜  vim security-ctx-web.yaml

student-node ~ ➜  cat security-ctx-web.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: ckad05-multi-pod-aecs
  namespace: ckad05-securityctx-aecs
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     securityContext:
      runAsUser: 1002
     command: ["sleep", "5000"]
  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]

student-node ~ ➜ k replace -f security-ctx-web.yaml --force 
pod "ckad05-multi-pod-aecs" deleted
pod/ckad05-multi-pod-aecs replaced

student-node ~ ➜  k exec -i -n ckad05-securityctx-aecs ckad05-multi-pod-aecs -c web -- id
uid=1002 gid=0(root) groups=0(root)

student-node ~ ➜  k exec -i -n ckad05-securityctx-aecs ckad05-multi-pod-aecs -c sidecar -- id
uid=1001 gid=0(root) groups=0(root)
```

</details>

**Details (Verification):**

* Is the "web" container configured correctly ?

-----

### Question 18

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a service account named `chatbot-service-account` in the namespace `ckad18-research-ns`.
  
Grant the service account `get` and `list` permissions to access **all resources** within the namespace using a Role named `chatbot-service-role`.
  
Also bind the **Role** to the service account using a **RoleBinding** named `chatbot-service-rolebinding`, restricting the access to the **ckad18-research-ns** namespace only.

`Note`: If the resources do not exist, please create them as well.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl create ns ckad18-research-ns
namespace/ckad18-research-ns created

student-node ~ ➜  kubectl create serviceaccount chatbot-service-account -n ckad18-research-ns
serviceaccount/chatbot-service-account created

student-node ~ ➜  kubectl create role chatbot-service-role --namespace=ckad18-research-ns --verb=get,list --resource=* 
role.rbac.authorization.k8s.io/chatbot-service-role created

student-node ~ ➜  kubectl create rolebinding chatbot-service-rolebinding \
   --role=chatbot-service-role \
   --serviceaccount=ckad18-research-ns:chatbot-service-account \
   --namespace=ckad18-research-ns
rolebinding.rbac.authorization.k8s.io/chatbot-service-rolebinding created
```

</details>

**Details (Verification):**

* Is service account created?
* Are roles created correctly?
* Is correct role defined for rolebinding?
* Is correct Service account specified for rolebinding?

-----

## Application Observability And Maintenance

### Question 19

**Context:** `kubectl config use-context cluster2`

**Task:**
A manifest file located at `root/ckad-flash89.yaml` on host **student-node**. Which can be used to create a multi-containers pod.

There are issues with the manifest file, preventing resource creation. Identify the errors, fix them and create resource.

<details>
<summary><b>View Solution</b></summary>

Set context to `cluster2`

use the kubectl create command we will see following error.

```bash
kubectl create -f ckad-flash89.yaml
error: resource mapping not found for name: "ckad-flash-89" namespace: "" from "ckad-flash89.yaml": no matches for kind "Pod" in version "V1"
ensure CRDs are installed first
```

about error shows us there is something wrong with apiVersion. So change it `v1` and try again. and check status.

```bash
kubectl get pods
NAME            READY   STATUS             RESTARTS      AGE
ckad-flash89-aom   1/2     CrashLoopBackOff   3 (39s ago)   93s
```

Now check for reason using

```bash
kubectl describe pod ckad-flash89-aom
```

we will see that there is problem with **nginx container**
  
 open yaml file and check in **spec -> nginx container** you can see error with `mountPath` --> **mountPath: "/var/log"** change it to **mountPath: `/var/log/nginx`** and apply changes.

</details>

**Details (Verification):**

* Pod status
* Used correct apiVersion
* Configured correct mount path

-----

### Question 20

**Task:**
Identify the Kubernetes API resources that correspond to `api_version=authorization.k8s.io/v1` by utilizing the kubectl command line interface. Store the results in the file `/root/api-version.txt` on the student node, ensuring that no headers are included in the output.

<details>
<summary><b>View Solution</b></summary>

Use the following command to get details:

```bash
kubectl api-resources --api-group=authorization.k8s.io  --no-headers > /root/api-version.txt
```

</details>

**Details (Verification):**

* apiVersion: authorization.k8s.io/v1
* Resources identified

-----

### Question 21

**Context:** `kubectl config use-context cluster1`

**Task:**
View the metrics (CPU and Memory) of the node `cluster1-node02` and copy the output to the `/root/cluster1-metrics` file in `clustername,CPU and memory`.

<details>
<summary><b>View Solution</b></summary>

Use the following command to get details:

```bash
kubectl top nodes cluster1-node02 > /root/cluster1-metrics
```

</details>

**Details (Verification):**

* Output saved to a file

-----
