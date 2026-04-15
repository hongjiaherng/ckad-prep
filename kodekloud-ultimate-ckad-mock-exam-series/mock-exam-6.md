# Lab: CKAD Mock Exam 6


## Application Design And Build

### Question 1

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a `persistent volume` called `data-pv-ckad02-str` with the below properties:  
  
  
**-** Its capacity should be `128Mi`.  
  
**-** The volume type should be `hostpath` and path should be `/opt/data-pv-ckad02-str`.

Next, create a `persistent volume claim` called `data-pvc-ckad02-str` as per below properties:  
  
  
**-** Request `50Mi` of storage from `data-pv-ckad02-str` PV.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create desired Kubernetes objects:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv-ckad02-str
spec:
  capacity:
    storage: 128Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /opt/data-pv-ckad02-str
  storageClassName: manual

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc-ckad02-str
spec:
  storageClassName: manual
  volumeName: data-pv-ckad02-str
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

</details>

**Details (Verification):**

* Is `data-pv-ckad02-str` PV capacity is '128Mi'?
* Is `data-pv-ckad02-str` PV `hostpath` is correct?
* Is data-pvc-ckad02-str`PVC requests for`50Mi` of storage?
* Is `data-pvc-ckad02-str` PVC requests storage from `data-pv-ckad02-str` PV?

-----

### Question 2

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-job` namespace, create a cron job called `my-alarm` that prints current datetime. It must be scheduled to run every Sunday at midnight.

In case the container in pod failed for any reason, it should be restarted automatically.

Use `busybox:1.28` image to create job.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create cronjob:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-alarm
  namespace: ckad-job
spec:
  schedule: "0 0 * * 0"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date;
          restartPolicy: OnFailure
```

</details>

**Details (Verification):**

* Is cronjob `my-alarm` created?
* Is the container image `busybox:1.28`?
* Does cronjob run the `date` command?
* Does cronjob runs every Sunday at midnight?

-----

### Question 3

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-pod-design` namespace, start a `ckad-httpd-fzkkvrwgms` pod runs `nginx:1.17` image; the container should be named `nginx-custom-environment`.  
  
  
Set environment variable in the default container as below:
```bash
EXAM: CKAD
```

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ckad-httpd-fzkkvrwgms
  name: ckad-httpd-fzkkvrwgms
  namespace: ckad-pod-design
spec:
  containers:
  - env:
    - name: EXAM
      value: CKAD
    image: nginx:1.17
    name: nginx-custom-environment
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.  
  
Alternatively, you can use this command for similar outcome:
```bash
kubectl run ckad-httpd-fzkkvrwgms -n ckad-pod-design --image=nginx:1.17 --env EXAM=CKAD
```

</details>

**Details (Verification):**

* Is the pod `ckad-httpd-fzkkvrwgms` running?
* Is the pod's container named `nginx-custom-environment`?
* Is the pod's container environment variable configured correctly?

-----

### Question 4

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad-pod-design` namespace, create a pod named `security-context-pod` that runs the `redis` image, and the container should be run in `privileged` mode.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create the desired pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: security-context-pod
  name: security-context-pod
  namespace: ckad-pod-design
spec:
  containers:
  - image: redis
    name: security-context-pod
    resources: {}
    securityContext:
      privileged: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Or in short, use this kubectl command for the same output:
```bash
kubectl run security-context-pod --image=redis --privileged=true -n ckad-pod-design
```

</details>

**Details (Verification):**

* Is pod `security-context-pod` running?
* Is the container image is `redis`?
* Is the container in privileged mode?

-----

### Question 5

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad-multi-containers` namespace, create a pod named `static-web-server`, which consists of 2 containers. One main container and one init-container both are running `alpine` image.   
  
  
Init container should print this message `Getting main application ready!` and then sleep for `10` seconds.  
  
Main container should print this message `Main application is running` and then sleep for `3600` seconds.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create required pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web-server
  namespace: ckad-multi-containers
  labels:
    app.kubernetes.io/name: static-web-server
spec:
  containers:
    - name: server-container
      image: alpine
      command:
        - sh
        - -c
        - echo Main application is running && sleep 3600
  initContainers:
    - name: init-myservice
      image: alpine
      command:
        - sh
        - -c
        - echo Getting main application ready! && sleep 10
```

</details>

**Details (Verification):**

* Is the `static-web-server` running?
* Does main container run `alpine` image?
* Does main container run the desired command?
* Does init container run `alpine` image?
* Does init container run the desired command?

-----


## Application Deployment

### Question 6

**Context:** `kubectl config use-context cluster3`

**Task:**
Our new client wants to deploy the resources through the popular Helm tool. In the initial phase, our team lead wants to deploy `nginx`, a very powerful and versatile web server software that is widely used to serve static content, reverse proxying, load balancing, from the `bitnami` helm chart on the `cluster3-controlplane` node.   
  
The chart URL and other specifications are as follows: -

**1.**  The chart URL link - `https://charts.bitnami.com/bitnami`

**2.**  The chart repository name should be `polar`.

**3.**  The release name should be `nginx-server`.

**4.**  All the resources should be deployed on the `cd-tool-apd` namespace.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster3
```

In this task, we will use the `helm` commands. Here are the steps: -

Add the repostiory to Helm with the following command: -
```bash
helm repo add polar https://charts.bitnami.com/bitnami
```

The `helm repo add` command is used to add a new chart repository to Helm and this allows us to browse and install charts from the new repository using the Helm package manager.

Use the `helm repo ls` command which is used to list all the currently configured Helm chart repositories on Kubernetes cluster.
```bash
helm repo ls
```

Search for the `nginx` chart in a `polar` chart repository as follows: -
```bash
helm search repo polar | grep nginx
```

The `helm search repo` command is used to search for charts in a specified Helm chart repository. Also, it allows us to browse available charts and view their details, such as their name, version, description, and maintainers.

Before installing the chart, we have to create a namespace as given in the task description. Then we can install the `nginx` chart on a Kubernetes cluster.
```bash
kubectl create ns cd-tool-apd

helm install nginx-server polar/nginx -n cd-tool-apd
```

</details>

**Details (Verification):**

* Is Helm repository created?
* Is Helm chart installed?
* Are resources deployed on ns?

-----

### Question 7

**Context:** `kubectl config use-context cluster3`

**Task:**
One of the Kubernetes developers from the team has deployed a test application named `test-v1-apd` to implement the blue/green deployment methodology.   
  
Now, the team wants to send `50%` of the traffic to the new test application, but since the main developer is away from the office, the team wants you to create a new deployment that can take the traffic from the pre-configured service. We have details of the application, which will help you create a new deployment: -

1. The new deployment name should be `test-v2-apd`.
2. Add label to a deployment `stage = test02`.
3. Use the image `kodekloud/webapp-color:v2` and the container name `test-v2`.
4. Scale a deployment in a way that the pre-configured service can send 50% of the traffic to this new deployment.

**NOTE: -**  We do not need to modify the `test-v1-apd` deployment.

You can test the application from the terminal by running the `curl` command with the following syntax: -
```bash
curl http://cluster3-controlplane:NODE-PORT
```

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster3
```

In this task, we will use the `kubectl` command. Here are the steps: -

1. Use the `kubectl get` command to list all the given resources: -
```bash
kubectl get deploy,svc -A
```

Here `-A` option lists the resources of all the namespaces.

2. Identify the deployment and service names.
3. Now, use the `kubectl create` command to create a deployment manifest file as follows: -
```bash
kubectl create deploy test-v2-apd --image=kodekloud/webapp-color:v2 --dry-run=client -oyaml > <FILE-NAME-1>.yaml
```

2. Open the file with any text editor such as `vi` or `nano` and update it as per the specifications. Check the labels for the pod template from the service. It should look like this: -
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    stage: test02
  name: test-v2-apd
spec:
  replicas: 3
  selector:
    matchLabels:
      test-app: v1
  template:
    metadata:
      labels:
        test-app: v1
    spec:
      containers:
      - image: kodekloud/webapp-color:v2
        name: test-v2
```

We got the instructions not to modify the `test-v1-apd` deployment, we must add the replica count to 3 for the `test-v2-apd` deployment. So we will deploy a total of 6 application pods for both deployments.   
  
Since the service distributes traffic to all pods equally, we have to set the replica count 3 to the `test-v2-apd` deployment so that the given service will send `50%` traffic to the deployment pods.

3. Now, create a deployment by using the `kubectl create -f` command: -
```bash
kubectl create -f <FILE-NAME-1>.yaml
```

</details>

**Details (Verification):**

* Is the deployment created?
* Is the deployment scaled correctly?
* Is deploymend configured correctly?

-----

### Question 8

**Context:** `kubectl config use-context cluster2`

**Task:**
On the `cluster2`, the team has installed multiple helm charts on a different namespace. By mistake, those deployed resources include one of the vulnerable images called `kodekloud/webapp-color:v1`. Find out the release name and uninstall it.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster2
```

In this task, we will use the `helm` commands and `jq` tool. Here are the steps: -

1. Run the `helm ls` command with `-A` option to list the releases deployed on all the namespaces using helm.
```bash
helm ls -A
```

2. We will use the `jq` tool to extract the image name from the deployments.
```bash
kubectl get deploy -n <NAMESPACE> <DEPLOYMENT-NAME> -o json | jq -r '.spec.template.spec.containers[].image'
```

Replace `<NAMESPACE>` with the namespace and `<DEPLOYMENT-NAME>` with the deployment name, which we get from the previous commands.

After finding the `kodekloud/webapp-color:v1` image, use the `helm uninstall` to remove the deployed chart that are using this vulnerable image.
```bash
helm uninstall <RELEASE-NAME> -n <NAMESPACE>
```

</details>

**Details (Verification):**

* Is helm release uninstalled?

-----

### Question 9

**Context:** `kubectl config use-context cluster3`

**Task:**
A web application running on cluster3 called `webserver-deployment` on the `production` namespace. The Ops team has created a new service account with a set of permissions for this web application. Update the newly created SA for this deployment.   
  
  
Also, change the strategy type to `Recreate`, so it will delete all the pods immediately and update the newly created SA to all the pods.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster3
```

In this task, we will use the `kubectl` command. Here are the steps: -

1. Use the `kubectl get` command to list all the given resources: -
```bash
kubectl get po,deploy,sa,ns -n production
```

Here `-n` option stands for `namespace`, which is used to specify the namespace.

The above command will list all the resources from the `production` namespace.

2. Inspect the service account is used by the pods/deployment.
```bash
kubectl get deploy -n production webserver-deployment -oyaml
```

The deployment is using the default service account.

3. Now, use the `kubectl get` command to retrieves the YAML definition of a deployment named `webserver-deployment` and save it into a file.
```bash
kubectl get deploy -n production webserver-deployment -o yaml > <FILE-NAME>.yaml
```

4. Open a `VI` editor. Make the necessary changes and save it. It should look like this: -
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    global-kgh: webserver-deployment
  name: webserver-deployment
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      global-kgh: webserver-deployment
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        global-kgh: webserver-deployment
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: robox-container
      serviceAccountName: service-account-galaxy
```

5. Now, replace the resource with the following command:
```bash
kubectl replace -f <FILE-NAME>.yaml --force
```

The above command will delete the existing deployment and create a new one with changes in the given namespace.

</details>

**Details (Verification):**

* Is the service account updated?
* Is the strategy type updated?

-----

### Question 10

**Context:** `kubectl config use-context cluster3`

**Task:**
On cluster3, we are getting alerts for the resource consumption for the `ckad10-staging` namespace. A limit is set for the `ckad10-staging` namespace; if the limit is crossed, we will get the alerts.   
  
After inspecting the namespace, the team found that the deployment scaled to 8, which is wrong.   
  
The team wants you to decrease the deployment to 4.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster3
```

In this task, we will use the `kubectl` command. Here are the steps: -

1. Use the `kubectl get` command to list the pods and deployment on the `ckad10-staging` namespace.
```bash
kubectl get pods,deployment -n ckad10-staging
```

Here `-n` option stands for `namespace`, which is used to specify the namespace.

2. Inspect the deployed resources and count the number of pods.

3. Now, scaled down the deployment to `4` as follows:
```bash
kubectl scale deployment ultra-deployment -n ckad10-staging --replicas=4
```

The above command will scale down the number of replicas in a deployment.

</details>

**Details (Verification):**

* Is deployment scaled down?

-----


## Services And Networking

### Question 11

**Context:** `kubectl config use-context cluster3`

**Task:**
Please use the namespace `nginx-depl-svcn` for the following scenario.

Create a Deployment named `nginx-ckad10-svcn` using the `nginx` image with `2` replicas.  
Ensure that the Pods created by this Deployment are labeled with `app=nginx-ckad`.

Expose this Deployment via a ClusterIP Service named `nginx-ckad10-service-svcn` on port `80`.  
Apply the label `app=nginx-ckad` to the Service as well.

Create and apply a NetworkPolicy named `netpol-ckad-allow-svcn` that allows ingress traffic only from Pods labeled `criteria=allow` to the Pods labeled `app=nginx-ckad`.

<details>
<summary><b>View Solution</b></summary>

1. Apply the Deployment:
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ckad10-svcn
  namespace: nginx-depl-svcn
  labels:
    app: nginx-ckad
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-ckad
  template:
    metadata:
      labels:
        app: nginx-ckad
    spec:
      containers:
      - name: nginx
        image: nginx
EOF
```

2. Apply the Service:
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-ckad10-service-svcn
  namespace: nginx-depl-svcn
  labels:
    app: nginx-ckad
spec:
  type: ClusterIP
  selector:
    app: nginx-ckad
  ports:
  - port: 80
    targetPort: 80
EOF
```

3. Apply the NetworkPolicy:
```yaml
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol-ckad-allow-svcn
  namespace: nginx-depl-svcn
spec:
  podSelector:
    matchLabels:
      app: nginx-ckad
  ingress:
  - from:
    - podSelector:
        matchLabels:
          criteria: allow
  policyTypes:
  - Ingress
EOF
```

</details>

**Details (Verification):**

* Is deployment with `nginx` image created?
* Does deployment created have 2 replicas?
* Is service of type "ClusterIP"?
* Is service configured to use targetPort 80?
* Is network policy - `netpol-ckad-allow` created?
* Does policy allows traffic from pod with label `criteria=allow`?
* Pods without mentioned labels cannot access.

-----

### Question 12

**Context:** `kubectl config use-context cluster3`

**Task:**
We have deployed an application in the `ns-new-ckad` namespace. We also configured services, namely `frontend-ckad-svcn` and `backend-ckad-svcn`.   
  
  
However, there are some issues:

- `backend-ckad-svcn` is not able to access backend pods
  
- `frontend-ckad-svcn` is not accessible from backend pods
  
  
  
Inspect them and troubleshoot the issues.

<details>
<summary><b>View Solution</b></summary>

- backend-ckad-svcn is not able to access backend pods

* To troubleshoot it check the label selector of the service.
```bash
kubectl -n ns-new-ckad describe svc backend-ckad-svcn 
Name:              backend-ckad-svcn
Namespace:         ns-new-ckad
Labels:            app=backend
                   tier=ckad-exam
Annotations:       <none>
Selector:          app=back-end,tier=ckadexam
----
```

* The labels used are not matching to the pod labels.
* Change them using `kubectl -n ns-new-ckad edit svc backend-ckad-svcn`.
```yaml
    protocol: TCP
    targetPort: 80
  selector:
    app: backend #change as per the pod
    tier: ckad-exam # change as per the pod
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

* Perform the connectivity test using `kubectl -n ns-new-ckad exec testpod -- curl backend-ckad-svcn`.

- frontend-ckad-svcn is not accessible from backend pods

* See if there are any Network policies restricting the traffic.
* Edit the policy using `kubectl -n ns-new-ckad edit netpol backend-egress-restricted`.
```yaml
namespace: ns-new-ckad
  resourceVersion: "4834"
  uid: a123fb01-4c83-4f3d-aa4e-55578589b654
spec:
  egress:
  - {}  # add this
  - to:  #remove this
    - podSelector: #remove this
        matchLabels: #remove this
          app: frontend #remove this
          tier: ckad-exam #remove this
  podSelector:
    matchLabels:
      app: backend
      tier: ckad-exam
  policyTypes:
  - Egress
```

</details>

**Details (Verification):**

* Can service `backend-ckad-svcn` access backend pods?
* Can `frontend-ckad-svcn` be accessed from backend pods?

-----

### Question 13

**Context:** `kubectl config use-context cluster1`

**Task:**
Deploy a pod with name `messaging-ckad04-svcn` using the `redis:alpine` image with the label `tier=msg`.  
  
  
  
Now, Create a service `messaging-service-ckad04-svcn` to expose the pod `messaging-ckad04-svcn` application within the cluster on port `6379`.

<details>
<summary><b>View Solution</b></summary>

Switch to `cluster1` :
```bash
kubectl config use-context cluster1
```

On student-node, use the command `kubectl run messaging-ckad04-svcn --image=redis:alpine -l tier=msg`  
  
  
  
Now run the command: `kubectl expose pod messaging-ckad04-svcn --port=6379 --name messaging-service-ckad04-svcn`.

</details>

**Details (Verification):**

* Pod Name: `messaging-ckad04-svcn`.
* Image used is `redis:alpine`.
* Pod is created with label `tier=msg`.
* Is service `messaging-service-ckad04-svcn` created?
* Is port `6379` configured for service?
* Is service created is of type "ClusterIP"?
* Does the service created use the right labels?

-----

### Question 14

**Context:** `kubectl config use-context cluster2`

**Task:**
This scenario is categorized into two parts. Please find them below.
  
  
**Part I**:  
  
We have already deployed several pods in the default namespace. Create a `ClusterIP` service .i.e. `radioactive-service`, which should expose the pods, namely, `beta` and `gamma`, with port set to `8080` and targetport to `80`.
  
  
**Part II**:
  
Store the **pod names** and their **ip addresses** from `all namespaces` at `/root/pod_ips_ckad02_svcn` where the output is sorted by their IPs.

<details>
<summary><b>View Solution</b></summary>

Switching to `cluster2`:
```bash
kubectl config use-context cluster2
```

The easiest way to route traffic to a specific pod is by the use of `labels` and `selectors` . List the pods along with their labels:
```bash
student-node ~ ➜  kubectl get pods --show-labels 
NAME     READY   STATUS    RESTARTS   AGE     LABELS
lambda   1/1     Running   0          5m21s   env=dev,mode=standard,type=external
alpha   1/1     Running   0          5m20s   env=dev,mode=standard,type=internal
delta   1/1     Running   0          5m20s   env=prod,mode=exam,type=internal
beta   1/1     Running   0          5m21s   env=dev,mode=exam,type=external
sigma   1/1     Running   0          5m20s   env=prod,mode=standard,type=internal
gamma   1/1     Running   0          5m20s   env=prod,mode=exam,type=external
```

Looks like there are a lot of pods created to confuse us. But we are only concerned with the labels of `beta` and `gamma`.   
  
  
  
As we can see both the required pods have labels `mode=exam,type=external` in common. Let's confirm that using kubectl too:
```bash
student-node ~ ➜  kubectl get pod -l mode=exam,type=external                                       
NAME     READY   STATUS    RESTARTS   AGE
beta   1/1     Running   0          9m18s
gamma   1/1     Running   0          9m17s
```

Nice!! Now as we have figured out the labels, we can proceed further with the creation of the service:
```bash
student-node ~ ➜  kubectl create service clusterip radioactive-service --tcp=8080:80 --dry-run=client -o yaml > radioactive-service.yaml
```

Now modify the service definition with selectors as required before applying to k8s cluster:
```bash
student-node ~ ➜  cat radioactive-service.yaml 
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: radioactive-service
  name: radioactive-service
spec:
  ports:
  - name: 8080-80
    port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: radioactive-service  # delete 
    mode: exam    # add
    type: external  # add
  type: ClusterIP
status:
  loadBalancer: {}
```

Finally let's apply the service definition:
```bash
student-node ~ ➜  kubectl apply -f radioactive-service.yaml
service/service-3421 created

student-node ~ ➜  k get ep radioactive-service 
NAME           ENDPOINTS                     AGE
service-3421   10.42.0.15:80,10.42.0.17:80   52s
```

To store all the pod name along with their IP's , we could use imperative command as shown below:
```bash
student-node ~ ➜  kubectl get pods -A -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP

POD_NAME                                  IP_ADDR
helm-install-traefik-crd-lbwzr            10.42.0.2
local-path-provisioner-7b7dc8d6f5-d4x7t   10.42.0.3
metrics-server-668d979685-vh7bk           10.42.0.4
...

# store the output to /root/pod_ips_ckad02_svcn
student-node ~ ➜  kubectl get pods -A -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP > /root/pod_ips_ckad02_svcn
```

</details>

**Details (Verification):**

* Does service `radioactive-service` exists ?
* Is "ClusterIP" the service `radioactive-service` type ?
* Is 8080 the service `radioactive-service` port ?
* Is 80 the service `radioactive-service` target port?
* Does service only expose pod "beta" and "gamma" ?
* Is the correct output stored in `/root/pod_ips_ckad02_svcn` ?

-----

### Question 15

**Context:** `kubectl config use-context cluster2`

**Task:**
We have deployed a pod `pod15-ckad-pod` in the default namespace. Create a service `svc15-ckad-service` that will expose the pod at port `443`.

<details>
<summary><b>View Solution</b></summary>

To use the cluster 3, switch the context using:
```bash
kubectl config use-context cluster2
```

To expose the pod `pod15-ckad-pod` at port 443, we can use the following imperative command.
```bash
kubectl expose pod pod15-ckad-pod --name=svc15-ckad-service --port=443
```

- It will create a service with name `svc15-ckad-service` and pod will be exposed at port 443.

</details>

**Details (Verification):**

* Is service `svc15-ckad-service` created?
* Is the pod exposed at port `443`?

-----


## Application Environment,  Configuration And Security

### Question 16

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a service account named `ckad23-sa-aecs` in the namespace `ckad23-nssa-aecs`.   
  
Grant the service account `get` and `list` permissions to access **all resources** within the namespace using a Role named `wide-access-aecs`.   
  
Also bind the **Role** to the service account using a **RoleBinding** named `wide-access-rb-aecs`, restricting the access to the **ckad23-nssa-aecs** namespace only.

`Note`: If the resources do not exist, please create them as well.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster3
Switched to context "cluster3".

student-node ~ ➜  kubectl create ns ckad23-nssa-aecs
namespace/ckad23-nssa-aecs created

student-node ~ ➜  kubectl create serviceaccount ckad23-sa-aecs -n ckad23-nssa-aecs
serviceaccount/ckad23-sa-aecs created

student-node ~ ➜  kubectl create role wide-access-aecs --namespace=ckad23-nssa-aecs --verb=get,list --resource=* 
role.rbac.authorization.k8s.io/wide-access-aecs created

student-node ~ ➜  kubectl create rolebinding wide-access-rb-aecs \
   --role=wide-access-aecs \
   --serviceaccount=ckad23-nssa-aecs:ckad23-sa-aecs \
   --namespace=ckad23-nssa-aecs
rolebinding.rbac.authorization.k8s.io/wide-access-rb-aecs created
```

</details>

**Details (Verification):**

* Is service account created?
* Are roles created correctly?
* Is correct role defined for rolebinding?
* Is correct Service account specified for rolebinding?

-----

### Question 17

**Context:** `kubectl config use-context cluster1`

**Task:**
Using the pod template on **student-node** at `/root/ckad08-dotfile-aecs.yaml` , create a pod `ckad08-top-secret-pod-aecs` in the namespace `ckad08-tp-srt-aecs` with the specifications as defined below:  
  
  
Define a volume section named `secret-volume` that is backed by a Kubernetes Secret named `ckad08-dotfile-secret-aecs`.  
  
  
Mount the `secret-volume` volume to the container's `/etc/secret-volume` directory in `read-only` mode, so that the container can access the secrets stored in the `ckad08-dotfile-secret-aecs` secret.

<details>
<summary><b>View Solution</b></summary>

```bash
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ckad08-top-secret-pod-aecs
  namespace: ckad08-tp-srt-aecs
spec:
  restartPolicy: Never
  volumes:
  - name: secret-volume
    secret:
      secretName: ckad08-dotfile-secret-aecs
  containers:
  - name: ckad08-top-scrt-ctr-aecs
    image: registry.k8s.io/busybox
    command:
    - ls
    - "-al"
    - "/etc/secret-volume"
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
EOF
```

</details>

**Details (Verification):**

* Is "read-only" volume created using "ckad08-dotfile-secret-aecs" ?
* Is 'readonly' volumeMount used with correct path ?

-----

### Question 18

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a service account `ckad14-sacrte-aecs` in the namespace `ckad14-sacrte-ns-aecs`.

<details>
<summary><b>View Solution</b></summary>

We can use the below command to create the service account.  
  
`kubectl create serviceaccount ckad14-sacrte-aecs -n ckad14-sacrte-ns-aecs`

</details>

**Details (Verification):**

* Is service account created ?

-----

### Question 19

**Context:** `kubectl config use-context cluster2`

**Task:**
We have a sample CRD at `/root/ckad19-crd-aecs.yaml` which should have the following validations:

- `destinationName`, `country`, and `city` must be **string** types.
  
- `pricePerNight` must be an **integer** between `50` and `5000`.
  
- `durationInDays` must be an **integer** between `1` and `30`.

Update the file incorporating the above validations in a `namespaced` scope.

`Note`: Remember to create the CRD after the required changes.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  vim ckad19-crd-aecs.yaml 

student-node ~ ➜  cat ckad19-crd-aecs.yaml 
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: holidaydestinations.destinations.k8s.io
  annotations:
    "api-approved.kubernetes.io": "unapproved, experimental-only"
  labels:
    app: holiday
spec:
  group: destinations.k8s.io
  names:
    kind: HolidayDestination
    singular: holidaydestination
    plural: holidaydestinations
    shortNames:
      - hd
  scope: Namespaced
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        # schema used for validation
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                destinationName:
                  type: string
                country:
                  type: string
                city:
                  type: string
                pricePerNight:
                  type: integer
                  minimum: 50
                  maximum: 5000
                durationInDays:
                  type: integer
                  minimum: 1
                  maximum: 30
            status:
              type: object
              properties:
                availableRooms:
                  type: integer
                  minimum: 0
                  maximum: 1000
      # subresources for the custom resource
      subresources:
        # enables the status subresource
        status: {}

student-node ~ ➜  k create -f ckad19-crd-aecs.yaml
customresourcedefinition.apiextensions.k8s.io/holidaydestinations.destinations.k8s.io created
```

</details>

**Details (Verification):**

* Is the "Namespaced" scope configured ?
* Are correct keys of type "string" defined?
* Are correct keys of type "integer" defined?
* Is the correct range for key 'durationInDays' defined?
* Is the correct range for key 'pricePerNight' defined?

-----


## Application Observability And Maintenance

### Question 20

**Context:** `kubectl config use-context cluster1`

**Task:**
Use the manifest file `/root/resourcedefinition.yaml` and extend kubernetes API for **mongodb** service.   
  
But there is an error with the file content. Find the error and fix the issue.

<details>
<summary><b>View Solution</b></summary>

Following error will be shown for creation of CRD
```bash
error: unable to recognize "resourcedefinition.yaml": no matches for kind "CustomResourceDefinition" in version "k8s.io/v1"
```

Check for apiVersion
```bash
kubectl api-resources | grep -i crd
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1
```

Replace the correct apiVersion
```bash
apiVersion: apiextensions.k8s.io/v1
```

and extend api with `kubectl create -f resourcedefinition.yaml`

</details>

**Details (Verification):**

* CRD created

-----

### Question 21

**Context:** `kubectl config use-context cluster1`

**Task:**
A pod called `log-generator-pod` is running in the default namespace. It has two co-located containers; get the logs of the `sidecar` container and copy them to `/root/ckad21-exam.txt` on student-node.

<details>
<summary><b>View Solution</b></summary>

Identify the containers name in **log-generator-pod** with
```bash
kubectl get pods log-generator-pod -o json | jq '.spec.containers[].name'
"ckad-exam"
"sidecar"
```

Use following command to logs of sidecar container
```bash
kubectl logs log-generator-pod -c sidecar > /root/ckad21-exam.txt
```

</details>

**Details (Verification):**

* Logs present

-----
