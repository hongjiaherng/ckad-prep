# Lab: CKAD Mock Exam 5


## Application Design And Build

### Question 1

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-pod-design` namespace, start a `ckad-httpd-bwutlljzof` pod running the `httpd:alpine` image.

The pod's container should be exposed at port `8080`.

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ckad-httpd-bwutlljzof
  name: ckad-httpd-bwutlljzof
  namespace: ckad-pod-design
spec:
  containers:
  - image: httpd:alpine
    name: ckad-httpd-bwutlljzof
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.  
  
Alternatively, you can use this command for similar outcome:
```bash
 kubectl run ckad-httpd-bwutlljzof --image=httpd:alpine --port=8080 -n ckad-pod-design
```

</details>

**Details (Verification):**

* Is pod `ckad-httpd-bwutlljzof` running?
* Is the container image `httpd:alpine`?
* Is pod's container exposed at `8080` port?

-----

### Question 2

**Context:** `kubectl config use-context cluster1`

**Task:**
A persistent volume called `papaya-pv-ckad09-str` is already created with a storage capacity of `150Mi`. It's using the `papaya-stc-ckad09-str` storage class with the path `/opt/papaya-stc-ckad09-str`.  
  
  
Also, a persistent volume claim named `papaya-pvc-ckad09-str` has been created on this cluster. This PVC has requested `50Mi` of storage from `papaya-pv-ckad09-str` volume.  
  
  
Resize the PVC to `80Mi` and make sure the PVC is in `Bound` state.

<details>
<summary><b>View Solution</b></summary>

Edit `papaya-pv-ckad09-str` PV:
```bash
kubectl get pv papaya-pv-ckad09-str -o yaml > /tmp/papaya-pv-ckad09-str.yaml
```

Edit the template:
```bash
vi /tmp/papaya-pv-ckad09-str.yaml
```

Delete all entries for `uid:`, `annotations`, `status:`, `claimRef:` from the template.

Edit `papaya-pvc-ckad09-str` PVC:
```bash
kubectl get pvc papaya-pvc-ckad09-str -o yaml > /tmp/papaya-pvc-ckad09-str.yaml
```

Edit the template:
```bash
vi /tmp/papaya-pvc-ckad09-str.yaml
```

Under `resources:` -> `requests:` change `storage: 50Mi` to `storage: 80Mi` and save the template.

Delete the exsiting PVC:
```bash
kubectl delete pvc papaya-pvc-ckad09-str
```

Delete the exsiting PV and create using the template:
```bash
kubectl delete pv papaya-pv-ckad09-str
kubectl apply -f /tmp/papaya-pv-ckad09-str.yaml
```

Create the PVC using template:
```bash
kubectl apply -f /tmp/papaya-pvc-ckad09-str.yaml
```

</details>

**Details (Verification):**

* Is `papaya-pvc-ckad09-str` PVC resized?

-----

### Question 3

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-job` namespace, create a job called `my-simple-job` that runs `date` command inside the container; use `busybox` image for this task.

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: my-simple-job
  namespace: ckad-job
spec:
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - date
        image: busybox
        name: my-simple-job
        resources: {}
      restartPolicy: Never
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.  
  
Alternatively, you can use this command for similar outcome:
```bash
kubectl create job my-simple-job --image=busybox -n ckad-job -- date
```

</details>

**Details (Verification):**

* Is the job `my-simple-job` created?
* Is the container image is `busybox`?
* Does the job run the `date` command?

-----

### Question 4

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a `persistent volume` called `cloudstack-pv` with the below properties:  
  
  
**-** Its capacity should be `128Mi`.  
  
**-** The volume type should be `hostpath` and path should be `/opt/cloudstack-pv`.

Next, create a `persistent volume claim` called `cloudstack-pvc` as per below properties:  
  
  
**-** Request `50Mi` of storage from `cloudstack-pv` PV.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create desired Kubernetes objects:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cloudstack-pv
spec:
  capacity:
    storage: 128Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /opt/cloudstack-pv
  storageClassName: manual

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cloudstack-pvc
spec:
  storageClassName: manual
  volumeName: cloudstack-pv
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

</details>

**Details (Verification):**

* Is `cloudstack-pv` PV capacity is '128Mi'?
* Is `cloudstack-pv` PV `hostpath` is correct?
* Is cloudstack-pvc`PVC requests for`50Mi` of storage?
* Is `cloudstack-pvc` PVC requests storage from `cloudstack-pv` PV?

-----

### Question 5

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad-job` namespace, schedule a job called `learning-every-hour` that prints this message in the shell every hour at 0 minutes: `I will pass CKAD certification`.

In case the container in pod failed for any reason, it should be restarted automatically.

Use `alpine` image for the cronjob!

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  namespace: ckad-job
  name: learning-every-hour
spec:
  schedule: "0 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: learning-every-hour
            image: alpine
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - echo I will pass CKAD certification
          restartPolicy: OnFailure
```

Then use `kubectl apply -f file_name.yaml` to create the required object.

</details>

**Details (Verification):**

* Is the cronjob `learning-every-hour` created?
* Is the container image `alpine`?
* Does cronjob run required command?
* Does cronjob run every hour?

-----


## Application Deployment

### Question 6

**Context:** `kubectl config use-context cluster2`

**Task:**
An application called `results-apd` is running on cluster2. In the weekly meeting, the team decides to upgrade the version of the existing image to `1.23.3` and wants to store the new version of the image in a file `/root/records/new-image-records.txt` on the `cluster2-controlplane` instance.

After upgrading the image version, to increase the availability and performance of the application, scale the deployment to `4`, and ensure that the targeted pod is running.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster2
```

In this task, we will use the `kubectl describe`, `kubectl get`, `kubectl set` and `kubectl scale` commands. Here are the steps: -

1. To check all the deployments in all the namespaces in the `cluster2`, we would have to run the following command:
```bash
kubectl get deployments -A
```

Inspect all the deployments.

2. We can see that one of the deployment's names is `results-apd` and deployed on `dashboard-apd` namespace. Use the `kubectl describe` command to get detailed information of that deployment: -
```bash
kubectl describe -n dashboard-apd deploy results-apd
```

The output of the `kubectl describe` command will provide you with a detailed description of the deployment, including its name, namespace, creation time, labels, replicas, and the Docker image being used.

3. In the previous command, we can see the container and image name under the `Pod Template` spec. Use the `kubectl set` command to update the image of that container as follows:
```bash
kubectl set image -n dashboard-apd deploy results-apd results-apd-container=nginx:1.23.3
```

After running the above command, Kubernetes will automatically update the `results-apd-container` container with the new image, and create a new replica of the resource with the updated image.   
The old replica will be deleted once the new one is up and running.

4. Now, `SSH` to the `cluster2-controlplane` node and use `echo` command to add this new image to a file at `/root/records/new-image-records.txt`: -
```bash
echo "nginx:1.23.3" > /root/records/new-image-records.txt
```

If the `records` directory is absent, use the `mkdir` command to create this directory.

> NOTE: - To exit from any node, type `exit` on the terminal or press `CTRL + D`.

5. Now, run the `kubectl scale` command to scale the deployment to `4`:
```bash
kubectl scale deployment -n dashboard-apd results-apd --replicas=4
```

Cross-verify the scaled deployment by using the `kubectl get` command:
```bash
kubectl get deployments,pods -n dashboard-apd
```

</details>

**Details (Verification):**

* Is the image updated?
* Is the deployment scaled?
* Did the image get stored in a file?

-----

### Question 7

**Context:** `kubectl config use-context cluster3`

**Task:**
One co-worker deployed an nginx helm chart on the `cluster3` server called `lvm-crystal-apd`. A new update is pushed to the helm chart, and the team wants you to update the helm repository to fetch the new changes.   
   
  
After updating the helm chart, upgrade the helm chart version to `18.1.15` and increase the replica count to `2`.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster3
```

In this task, we will use the `kubectl` and `helm` commands. Here are the steps: -

1. Log in to the `cluster3-controlplane` node first and use the `helm ls` command to list all the releases installed using Helm in the Kubernetes cluster.
```bash
helm ls -A
```

Here `-A` or `--all-namespaces` option lists all the releases of all the namespaces.

2. Identify the namespace where the resources get deployed.

3. Use the `helm repo ls` command to list the helm repositories.
```bash
helm repo ls 
```

4. Now, update the helm repository with the following command: -
```bash
helm repo update lvm-crystal-apd -n crystal-apd-ns
```

The above command updates the local cache of available charts from the configured chart repositories.

5. The `helm search` command searches for all the available charts in a specific Helm chart repository. In our case, it's the nginx helm chart.
```bash
helm search repo lvm-crystal-apd/nginx -n crystal-apd-ns -l | head -n30
```

The `-l` or `--versions` option is used to display information about all available chart versions.

6. Upgrade the helm chart to `18.1.15` and also, increase the replica count of the deployment to `2` from the command line. Use the `helm upgrade` command as follows: -
```bash
helm upgrade lvm-crystal-apd lvm-crystal-apd/nginx -n crystal-apd-ns --version=18.1.15 --set replicaCount=2
```

7. After upgrading the chart version, you can verify it with the following command: -
```bash
helm ls -n crystal-apd-ns
```

Look under the `CHART` column for the chart version.

8. Use the `kubectl get` command to check the replicas of the deployment: -
```bash
kubectl get deploy -n crystal-apd-ns
```

The available count `2` is under the `AVAILABLE` column.

</details>

**Details (Verification):**

* Is deployment running?
* Is the chart version upgraded?
* Is the deployment scaled?

-----

### Question 8

**Context:** `kubectl config use-context cluster1`

**Task:**
One application, `webpage-server-01`, is deployed on the Kubernetes cluster by the Helm tool. Now, the team wants to deploy a new version of the application by replacing the existing one. A new version of the helm chart is given in the `/root/new-version` directory on the `student-node`. Validate the chart before installing it on the Kubernetes cluster.   
  
  
Use the `helm` command to validate and install the chart. After successfully installing the newer version, uninstall the older version.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster1
```

In this task, we will use the `helm` commands. Here are the steps: -

1. Use the `helm ls` command to list the release deployed on the `default` namespace using helm.
```bash
helm ls -n default
```

2. First, validate the helm chart by using the `helm lint` command: -
```bash
cd /root/

helm lint ./new-version
```

3. Now, install the new version of the application by using the `helm install` command as follows: -
```bash
helm install --generate-name ./new-version
```

We haven't got any release name in the task, so we can generate the random name from the `--generate-name` option.

Finally, uninstall the old version of the application by using the `helm uninstall` command: -
```bash
helm uninstall webpage-server-01 -n default
```

OR you can do a `helm upgrade` as well after linting:
```bash
helm upgrade webpage-server-01 /root/new-version
```

</details>

**Details (Verification):**

* Is the new version app deployed?
* Is the old version app uninstalled?

-----

### Question 9

**Context:** `kubectl config use-context cluster2`

**Task:**
One application deployment called `deluxe-apd` got deployed on the wrong namespace. The team wants you to take the appropriate steps to deploy it on the `newspace` namespace.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster2
```

In this task, we will use the `kubectl` command. Here are the steps: -

1. Use the `kubectl get` command to list the deployments on all the namespaces.
```bash
kubectl get deployment -A
```

Here `-A` option stands for `all namespaces`, which displays resources from all namespaces.

Inspect the deployment name `deluxe-apd` from the above command output and identify which namespace it got deployed.

1. Use the `kubectl get` command to retrieves the YAML definition of a deployment named `deluxe-apd` and save it into a file.
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
  namespace: newspace 
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

2. Before deploying on the given namespace, list the namespaces if it exists or not.
```bash
kubectl get ns
```

If it's not exist, then create the namespace first with the following command: -
```bash
kubectl create ns newspace
```

3. Now, deploy the resource with the following command:
```bash
kubectl create -f <FILE-NAME>.yaml
```

The above command will deploy the resource in the given namespace.

</details>

**Details (Verification):**

* Is app deployed on the correct ns?

-----

### Question 10

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `dev-apd` namespace, one of the developers has performed a rolling update and upgraded the application to a newer version. But somehow, application pods are not being created.

To regain the working state, `rollback` the application to the previous version.

After rolling the deployment back, on the `controlplane` node, save the image currently in use to the `/root/records/rolling-back-record.txt` file and increase the replica count to `3`.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster2
```

In this task, we will use the `kubectl describe`, `kubectl get`, `kubectl rollout` and `kubectl scale` commands. Here are the steps: -

1. First check the status of the pods: -
```bash
kubectl get pods -n dev-apd
```

One of the pods is in an error state. By using the `kubectl describe` command. We can see that there is an issue with the image.   
  
We can check the revision history of a deployment by using the `kubectl history` command as follows: -
```bash
kubectl rollout history -n dev-apd deploy webapp-apd  
```

Inspect the revision in detail as follows: -
```bash
kubectl rollout history -n dev-apd deploy webapp-apd --revision=2
```

We found that the image issue happened because of the wrong image tag, and the previous image was correct. As a quick fix, we need to roll back to the previous revision. Use the `kubectl rollout` command: -
```bash
kubectl rollout undo -n dev-apd deploy webapp-apd
```

After successful rolling back, inspect the updated image: -
```bash
kubectl describe deploy -n dev-apd webapp-apd | grep -i image
```

On the `Controlplane` node, save the image name to the given path `/root/records/rolling-back-record.txt`: -
```bash
ssh cluster2-controlplane

echo "kodekloud/webapp-color" > /root/records/rolling-back-record.txt
```

If the `records` directory is absent, use the `mkdir` command to create this directory.

> NOTE: - To exit from any node, type `exit` on the terminal or press `CTRL + D`.

And increase the replica count to the `3` with help of `kubectl scale` command: -
```bash
kubectl scale deploy -n dev-apd webapp-apd --replicas=3
```

Verify it by running the command: `kubectl get deploy -n dev-apd`

</details>

**Details (Verification):**

* Is rolling back successful?
* Is the Image saved to a file?
* Is the deployment scaled?

-----


## Services And Networking

### Question 11

**Context:** `kubectl config use-context cluster1`

**Task:**
For this scenario, create an ingress controller.

We have already deployed some of the required resources (Namespaces, Service accounts, Roles and Rolebindings).
  
  
Your task is to create the Ingress controller Deployment using the manifest given at `/root/nginx-controller.yaml`. There are some issues in the configuration. Please find the issues and fix them.

<details>
<summary><b>View Solution</b></summary>

Use cat command to view the contents of the file. `cat /root/nginx-controller.yaml`

- Please check the apiVersion, resource kind, namespace and the container port sections
```yaml
apiVersion: apps/betav1 #wrong api version
kind: deployment #change this
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.6.4
  name: ingress-nginx-controller
  namespace: ingressnginx  #issue1 ingress-nginx
spec:
  minReadySeconds: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --publish-service=/ingress-nginx-controller
        - --election-id=ingress-nginx-leader
        - --controller-class=k8s.io/ingress-nginx
        - --ingress-class=nginx
        - --configmap=/ingress-nginx-controller
        - --validating-webhook=:8443
        - --validating-webhook-certificate=/usr/local/certificates/cert
        - --validating-webhook-key=/usr/local/certificates/key
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: LD_PRELOAD
          value: /usr/local/lib/libmimalloc.so
        image: registry.k8s.io/ingress-nginx/controller:v1.6.4@sha256:15be4666c53052484dd2992efacf2f50ea77a78ae8aa21ccd91af6baaa7ea22f
        imagePullPolicy: IfNotPresent
        lifecycle:
          preStop:
            exec:
              command:
              - /wait-shutdown
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: controller
        ports:
        -    containerPort: 80 #wrong indentation 
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8443
          name: webhook
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          requests:
            cpu: 100m
            memory: 90Mi
        securityContext:
          allowPrivilegeEscalation: true
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - ALL
          runAsUser: 101
        volumeMounts:
        - mountPath: /usr/local/certificates/
          name: webhook-cert
          readOnly: true
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
      - name: webhook-cert
        secret:
          secretName: ingress-nginx-admission
```

`

</details>

**Details (Verification):**

* Is the deployment `ingress-nginx-controller` created?
* Is controller deployed in namespace `ingress-nginx`?
* Does deployment have 1 replica?

-----

### Question 12

**Context:** `kubectl config use-context cluster3`

**Task:**
We have deployed an application named `app-ckad-svcn` in the `default` namespace. Configure a service `multi-port-svcn` for the application which exposes the pods at multiple ports with different protocols.

- Expose port 80 using the TCP with name `http`
  
- Expose port 53 using the UDP with name `dns`

<details>
<summary><b>View Solution</b></summary>

The pod `app-ckad-svcn` is deployed in default namespace.

- To view the pod along with labels, use the following command.
```bash
student-node ~ ➜  kubectl get pods app-ckad-svcn --show-labels 
NAME            READY   STATUS    RESTARTS   AGE     LABELS
app-ckad-svcn   1/1     Running   0          2m58s   app=app-ckad,scenario=multiport
```

- We will use those labels to create the service.
  
- Create a service using the following manifest. It will create a service with multiple ports expose with different protocols.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-svcn
  labels:
       app: app-ckad
       scenario: multiport
spec:
  selector:
      app: app-ckad
      scenario: multiport
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 53
    targetPort: 53
    protocol: UDP
    name: dns
```

</details>

**Details (Verification):**

* Is port 80 exposed using TCP?
* Is port 53 exposed using UDP?
* Is service `multi-port-svcn` created?
* Are correct labels used?

-----

### Question 13

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a Kubernetes Service named `ckad14-svcn` that routes traffic to the external domain `my.application.test.com`. This service should be configured as type `ExternalName`.

<details>
<summary><b>View Solution</b></summary>

Create the service using the following manifest:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ckad14-svcn
spec:
  type: ExternalName
  externalName: my.application.test.com
```

</details>

**Details (Verification):**

* Is service `ckad14-svcn` created?
* Is service of type - "ExternalName"?
* Is service configured with externalName - `my.application.test.com`?

-----

### Question 14

**Context:** `kubectl config use-context cluster2`

**Task:**
A new payment service has been introduced. Since it is a sensitive application, it is deployed in its own namespace `sensitive-space`. Inspect the resources and service created.  
  
  
You are requested to make the new application available at `/pay`. Create an ingress resource named `ingress-ckad14-pay` for the payment application to make it available at `/pay`

<details>
<summary><b>View Solution</b></summary>

Switch to cluster2 using the following:
```bash
kubectl config use-context cluster2
```

Solution manifest file to create a new ingress service to make the application available at `/pay` as follows:
```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-ckad14-pay
  namespace: sensitive-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
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

* Is ingress `ingress-ckad14-pay` created?
* Is ingress configured for service `pay-service`?
* is the correct port configured?

-----


## Application Environment,  Configuration And Security

### Question 15

**Context:** `kubectl config use-context cluster1`

**Task:**
We have already deployed the required pods and services in the namespace `ckad01-appstk-sec-aecs`.

1. Create a new secret named `ckad01-db-scrt-aecs` with the data given below.  
     
     
   Secret Name: `ckad01-db-scrt-aecs`  
     
   Secret 1: `DB_Host=sql01`   
     
   Secret 2: `DB_User=root`  
     
   Secret 3: `DB_Password=password123`
2. Configure `ckad01-webapp-pod-aecs` to load environment variables from the newly created secret, where the **keys** from the secret should become the **environment variable name** in the Pod.

<details>
<summary><b>View Solution</b></summary>
```yaml
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  k get all -n ckad01-appstk-sec-aecs
NAME                         READY   STATUS    RESTARTS   AGE
pod/ckad01-webapp-pod-aecs   1/1     Running   0          3m13s
pod/ckad01-db-pod-aecs       1/1     Running   0          3m13s

NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/ckad01-webapp-service-aecs   NodePort    10.43.190.89    <none>        8080:30080/TCP   3m13s
service/ckad01-db-svc-aecs           ClusterIP   10.43.117.255   <none>        3306/TCP         3m13s

student-node ~ ➜  kubectl create secret generic ckad01-db-scrt-aecs \
   --namespace=ckad01-appstk-sec-aecs \
   --from-literal=DB_Host=sql01 \
   --from-literal=DB_User=root \
   --from-literal=DB_Password=password123
secret/ckad01-db-scrt-aecs created

student-node ~ ➜  k get -n ckad01-appstk-sec-aecs pod ckad01-webapp-pod-aecs -o yaml > webapp-pod-sec-cfg.yaml

student-node ~ ➜  vim webapp-pod-sec-cfg.yaml

student-node ~ ➜  cat webapp-pod-sec-cfg.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: ckad01-webapp-pod-aecs
  name: ckad01-webapp-pod-aecs
  namespace: ckad01-appstk-sec-aecs
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: ckad01-db-scrt-aecs

student-node ~ ➜  kubectl replace -f webapp-pod-sec-cfg.yaml --force 
pod "ckad01-webapp-pod-aecs" deleted
pod/ckad01-webapp-pod-aecs replaced

student-node ~ ➜  kubectl exec -n ckad01-appstk-sec-aecs ckad01-webapp-pod-aecs -- printenv | egrep -w 'DB_Password=password123|DB_User=root|DB_Host=sql01'
DB_Password=password123
DB_User=root
DB_Host=sql01
```

</details>

**Details (Verification):**

* Is the secret created with correct values ?
* Is the pod configured to use secret as env vars?

-----

### Question 16

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a role `configmap-updater` in the `ckad21-auth2-aecs` namespace granting the `update` and `get` permissions on `configmaps` resources but restricted to only the `ckad-cnfmp-aecs` instance of the resource.

<details>
<summary><b>View Solution</b></summary>
```yaml
student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  k get cm -n ckad21-auth2-aecs 
NAME               DATA   AGE
kube-root-ca.crt   1      3m35s
ckad-cnfmp-aecs    2      3m35s

student-node ~ ➜  kubectl create role configmap-updater --namespace=ckad21-auth2-aecs --resource=configmaps --resource-name=ckad-cnfmp-aecs --verb=update,get 
role.rbac.authorization.k8s.io/configmap-updater created

student-node ~ ➜  k get role -n ckad21-auth2-aecs configmap-updater -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2023-03-22T09:01:04Z"
  name: configmap-updater
  namespace: ckad21-auth2-aecs
  resourceVersion: "2799"
  uid: c152750a-198e-438e-9993-64b3e872c3e0
rules:
- apiGroups:
  - ""
  resourceNames:
  - ckad-cnfmp-aecs
  resources:
  - configmaps
  verbs:
  - update
  - get
```

</details>

**Details (Verification):**

* Are correct verbs defined?
* Are correct resource and resource instance defined ?

-----

### Question 17

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a `role` named `pod-reader` in the `ckad17-auth-ns` namespace, and grant only the **list**, **watch** and **get** permissions on `pods` resources.  
  
  
Create a `role binding` named `read-pods` in the same namespace, and assign the **pod-reader** role to a user named `jane`.

<details>
<summary><b>View Solution</b></summary>
```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl create ns ckad17-auth-ns

student-node ~ ➜ kubectl create role pod-reader --namespace=ckad17-auth-ns --verb=list,watch,get --resource=pods
role.rbac.authorization.k8s.io/pod-reader created

student-node ~ ➜  kubectl create rolebinding read-pods --namespace=ckad17-auth-ns --role=pod-reader --user=jane
rolebinding.rbac.authorization.k8s.io/read-pods created

# Now let's validate if our role and role binding is working as expected
student-node ~ ➜  kubectl auth can-i create pod --as jane
no

student-node ~ ✖ kubectl auth can-i create pod --as jane --namespace ckad17-auth-ns
yes
```

</details>

**Details (Verification):**

* Is correct resource for role "pod-reader" specified?
* Are correct "verbs" specified for the role?
* Is correct role used for rolebinding?
* Is correct user specified for rolebinding?

-----


## Application Observability And Maintenance

### Question 18

**Context:** `kubectl config use-context cluster1`

**Task:**
A template to create a Kubernetes pod is stored at `/root/probe-ckad-aom.yaml` on the `student-node`. Set the `initialDelaySeconds to 5`. However, using this template results in an error.  
  
  
Fix the issue with this template and use it to create the pod. Once created, watch the pod for a minute or two to make sure it is stable i.e., it is not crashing or restarting.

<details>
<summary><b>View Solution</b></summary>

##### Try to apply the template
```bash
kubectl apply -f probe-ckad-aom.yaml 
```

You will see error:
```bash
error: error validating "probe-ckad-aom.yaml": error validating data: [ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): unknown field "command" in io.k8s.api.core.v1.HTTPGetAction, ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): missing required field "port" in io.k8s.api.core.v1.HTTPGetAction]; if you choose to ignore these errors, turn validation off with --validate=false
```

From the error you can see that the error is for liveness probe, so let's open the template to find out:
```bash
vi probe-ckad-aom.yaml
```

Under `livenessProbe:` you will see the type is `httpGet` however the rest of the options are command based so this probe should be of `exec` type.

* Change `httpGet` to `exec`

##### Try to apply the template now
```bash
kubectl apply -f probe-ckad-aom.yaml 
```

Cool it worked, now let's watch the POD status, after few seconds you will notice that POD is restarting. So let's check the logs/events
```bash
kubectl get event --field-selector involvedObject.name=probe-ckad-aom
```

You will see an error like:
```bash
21s         Warning   Unhealthy   pod/red-probe-ckad-aom   Liveness probe failed: cat: can't open '/healthcheck': No such file or directory
```

So seems like Liveness probe is failing, lets look into it:
```bash
vi probe-ckad-aom.yaml
```

Notice the command `- sleep 3 ; touch /healthcheck; sleep 30;sleep 30000` it starts with a delay of `3` seconds, but the liveness probe `initialDelaySeconds` is set to `1` and `failureThreshold` is also `1`. Which means the POD will fail just after first attempt of liveness check which will happen just after `1` second of pod start. So to make it stable we must increase the `initialDelaySeconds` to at least `5`
```bash
vi probe-ckad-aom.yaml
```

* Change `initialDelaySeconds` from `1` to `5` and save apply the changes.

Delete old pod:
```bash
kubectl delete pod probe-ckad-aom
```

Apply changes:
```bash
kubectl apply -f probe-ckad-aom.yaml
```

</details>

**Details (Verification):**

* Probe type changed to exec ?
* Is POD stable?
* Initial delay seconds is set to 5 ?

-----

### Question 19

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a pod `resou-limit` using yaml file provided in the `/root/resou-limit-aom.yaml` .However, currently, there is an issue in the manifest. Find the issue, fix it and create the pod.

<details>
<summary><b>View Solution</b></summary>

You will see the following error
```bash
the Pod "resou-limit-aom" is invalid: spec.containers[0].resources.requests: Invalid value: "2": must be less than or equal to cpu limit
```

Update the yaml file
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resou-limit-aom
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    resources:
      requests:
        cpu: "1"
        memory: "256Mi"
      limits:
        cpu: "1"
        memory: "512Mi" 
```

Now create the pod using `kubectl create -f resou-limit-aom.yaml`

</details>

**Details (Verification):**

* Is pod `resou-limit-aom` created?
* CPU request set to 1

-----

### Question 20

**Context:** `kubectl config use-context cluster1`

**Task:**
A pod named `backend-pod` is deployed and exposed with a service `service-backend`, but it seems the service is not configured properly and is not selecting the correct pod.   
  
Make the required changes to service and ensure the endpoint is configured for service.

<details>
<summary><b>View Solution</b></summary>

Check for service end point by using
```bash
kubectl describe svc service-backend
```

we can see below output as below
```bash
kubectl describe svc service-backend
Name:              service-backend
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=ngnix
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.134.231
IPs:               10.43.134.231
Port:              80-80  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```

we can see there **endpoint** value as **none**. Let's debug this we can see **selector as app=ngnix** . Lets check label value in pod.
```bash
kubectl get pod backend-pod -o json | jq -r .metadata.labels
  "app": "nginx"
```

we can see here selector is **mis-spelled** in service. so edit service and check for endpoint value.
```bash
kubectl get ep service-backend
NAME                     ENDPOINTS                   AGE
service-backend   10.42.2.4:80,10.42.2.5:80   4m44s
```

</details>

**Details (Verification):**

* Is endpoint configured?

-----

### Question 21

**Context:** `kubectl config use-context cluster3`

**Task:**
A `ckad-frontend-pod` has been deployed in `cluster3` in the `ckad-21-production` namespace.   
  
There is a problem with the pod; it is not in running state find the cause for its present state and modify the content of the pod if required to bring it to the `running` state.

<details>
<summary><b>View Solution</b></summary>

Set the context to cluster 2
```bash
kubectl config use-context cluster3
```

Check for status of pod in `ckad-21-production` namespace
```bash
student-node ~ ➜  kubectl get pods -n ckad-21-production
NAME            READY   STATUS            RESTARTS   AGE
ckad-init-pod   0/1     Init:StartError   0          72s
```

Error is indicating that there is problem with init container.   
 use
```bash
kubectl describe pod ckad-frontend-pod -n ckad-21-production
```

and observe for following error
```bash
Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process
```

get the yaml file for the pod and add `/bin/sh` interpreter to the initcontainer command and now use this yaml to create a pod in `ckad-21-production namespace` on cluster 2.

</details>

**Details (Verification):**

* Modified init container command
* Pod status is running

-----
