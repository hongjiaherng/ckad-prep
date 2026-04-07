# Lab: CKAD Mock Exam 3

## Application Design And Build

### Question 1

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-job` namespace, create a cronjob named `simple-python-job` to run every 30 minutes to list all the running processes inside a container that used `python` image (the command needs to be run in a shell).

In Unix-based operating systems, `ps -eaf` can be use to list all the running processes.

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-python-job
  namespace: ckad-job
spec:
  schedule: "*/30 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: simple-python-job
            image: python
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - ps -eaf
          restartPolicy: OnFailure
```

Then use `kubectl apply -f file_name.yaml` to create the required object.

You can also use below command to create cronjob

```yaml
kubectl create cronjob simple-python-job -n ckad-job --image=python --schedule="*/30 * * * *" -- /bin/bash -c "ps -eaf"
```

</details>

**Details (Verification):**

* Is cronjob `simple-python-job` created?
* Is the container image `python`?
* Does cronjob run `ps -eaf` command?
* Does cronjob run every 30 minutes?

-----

### Question 2

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-pod-design` namespace, start a `ckad-nginx-uahsbcbdkl` pod running the `nginx:1.17` image.

Configure the pod with a label:

```
TRAINER: KODEKLOUD
```

The pod should not be restarted in any case if it has already exited.

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    TRAINER: KODEKLOUD
  name: ckad-nginx-uahsbcbdkl
  namespace: ckad-pod-design
spec:
  containers:
  - image: nginx:1.17
    name: ckad-nginx-uahsbcbdkl
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.  
  
Alternatively, you can use this command for similar outcome:

```yaml
kubectl run ckad-nginx-uahsbcbdkl --image=nginx:1.17 -n ckad-pod-design -l TRAINER=KODEKLOUD --restart=Never
```

</details>

**Details (Verification):**

* Is pod `ckad-nginx-uahsbcbdkl` running?
* Is the container image `nginx:1.17`?
* Is label configured correctly?
* Is Restart Policy configured correctly?

-----

### Question 3

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-pod-design` namespace, create a pod called `ckad-busybox-gsqodeuykj` that runs a `busybox:1.28` image.
  
The pod's container should be named `busybox-server`; the container will sleep for `3600` seconds.

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ckad-busybox-gsqodeuykj
  name: ckad-busybox-gsqodeuykj
  namespace: ckad-pod-design
spec:
  containers:
  - command:
    - sleep
    - "3600"
    image: busybox:1.28
    name: busybox-server
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.  
  
Alternatively, you can use this command for similar outcome:

```yaml
kubectl run ckad-busybox-gsqodeuykj --image=busybox:1.28 -n ckad-pod-design --command sleep 3600
```

</details>

**Details (Verification):**

* Is the pod `ckad-busybox-gsqodeuykj` running?
* Is the container image `busybox:1.28`?
* Is the required command available?

-----

### Question 4

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad-multi-containers` namespace, create a pod named `cuda-pod`, which has 2 containers matching the below requirements:

* The first container named `alpha` runs `alpine` image and has `release=stable` environment variable.
* The second container named `beta` runs `nginx:1.17` image and is exposed at port `8080`.

**NOTE**: All pod containers should be in the running state.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create desired pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    environment: dev
  name: cuda-pod
  namespace: ckad-multi-containers
spec:
  containers:
  - env:
    - name: release
      value: stable
    image: alpine
    name: alpha
    command:
    - /bin/sh
    - -c
    - sleep 3600;
    resources: {}
  - image: nginx:1.17
    name: beta
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</details>

**Details (Verification):**

* Is the `alpha` container running?
* Is the `alpha` container use `alpine` image?
* Is the `alpha` container environment variable well configured?
* Does the `beta` container running?
* Does the `beta` container use `nginx:1.17` image?
* Is the `beta` container expose at port `8080`?

-----

### Question 5

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-pod-design` namespace, we created a pod named `custom-nginx` that runs the `nginx:1.17` image.
  
Take appropriate actions to update the `index.html` page of this NGINX container with below value instead of default NGINX welcome page:

```
Welcome to CKAD mock exams!
```

**NOTE:** By default NGINX web server default location is at `/usr/share/nginx/html` which is located on the default file system of the Linux.

<details>
<summary><b>View Solution</b></summary>

Exec to the pod container and update the index.html file content:

```YAML
student-node ~ ➜kubectl exec -it -n ckad-pod-design custom-nginx -- sh
# echo 'Welcome to CKAD mock exams!' > /usr/share/nginx/html/index.html
```

Observe the result:

```YAML
student-node ~ ➜ kubectl exec -it -n ckad-pod-design custom-nginx -- cat /usr/share/nginx/html/index.html
Welcome to CKAD mock exams!
```

</details>

**Details (Verification):**

* Is pod `custom-nginx` running?
* Is the `index.html` file updated properly?

-----

## Application Deployment

### Question 6

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a deployment called `app-apd01` using the `nginx` image and scale the application pods to `3`.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster1
```

Run the following command: -

```bash
kubectl create deployment app-apd01 --image=nginx --replicas=3
```

To cross-verify the deployed resources, run the `kubectl get` command as follows: -

```bash
kubectl get pods,deployments
```

</details>

**Details (Verification):**

* Is deployment running?

-----

### Question 7

**Context:** `kubectl config use-context cluster2`

**Task:**
On the `student-node`, a Helm chart repository is given under the `/opt/` path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. The files have some issues. Fix those issues and deploy them with the following specifications: -

1. The release name should be `webapp-color-apd`.
2. All the resources should be deployed on the `frontend-apd` namespace.
3. The service type should be `node port`.
4. Scale the deployment to `3`.
5. Application version should be `1.20.0`.

`NOTE`: - Remember to make necessary changes in the `values.yaml` and `Chart.yaml` files according to the specifications, and, to fix the issues, inspect the template files.

You can start new terminal to open `student-node` command.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster2
```

In this task, we will use the `helm` commands. Here are the steps: -

1. First, check the given namespace; if it doesn't exist, we must create it first; otherwise, it will give an error **"namespaces not found"** while installing the helm chart.

To check all the namespaces in the `cluster2`, we would have to run the following command: -

```bash
kubectl get ns
```

It will list all the namespaces. If the given namespace doesn't exist, then run the following command: -

```bash
kubectl create ns frontend-apd
```

Now, on the `student-node` node and go to the `/opt/` directory. We have given the helm chart directory `webapp-color-apd` that contains templates, values files, and the chart file etc.

Update the values according to the given specifications as follows: -

a.) Update the value of the `appVersion` to `1.20.0` in the `Chart.yaml` file.

b.) Update the value of the `replicaCount` to `3` in the `values.yaml` file.

c.) Update the value of the `type` to `NodePort` in the `values.yaml` file.

These are the values we have to update.

Now, we will use the `helm lint` command to check the Helm chart because it can identify errors such as missing or misconfigured values, invalid YAML syntax, and deprecated APIs etc.

```bash
cd /opt/
helm lint ./webapp-color-apd/
```

If there is no misconfiguration, we will see the similar output: -

```bash
helm lint ./webapp-color-apd/
==> Linting ./webapp-color-apd/
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed
```

But in our case, there are some issues with the given `templates`.

1. Deployment apiVersion needs to be correctly written. It should be `apiVersion: apps/v1`.
2. In the service YAML, there is a typo in the template variable `{{ .Values.service.name }}` because of that, it's not able to reference the value of the name field defined in the `values.yaml` file for the Kubernetes service that is being created or updated.

Now run the following command to install the helm chart in the `frontend-apd` namespace: -

```bash
# Navigate to the directory having the chart
cd /opt
# Install the helm chart
helm install webapp-color-apd -n frontend-apd ./webapp-color-apd
```

Use the `helm ls` command to list the release deployed using helm.

```bash
helm ls -n frontend-apd
```

</details>

**Details (Verification):**

* Is the release name set?
* Are the resources deployed on given namespace?
* Is the service type defined?
* Is the deployment scaled?
* Is the application version set?
* Are resources running?

-----

### Question 8

**Context:** `kubectl config use-context cluster1`

**Task:**
In this task, we have to create two identical environments that are running different versions of the application. The team decided to use the Blue/green deployment method to deploy a total of 10 application pods which can mitigate common risks such as downtime and rollback capability.

Also, we have to route traffic in such a way that `30%` of the traffic is sent to the `green-apd` environment and the rest is sent to the `blue-apd` environment. All the development processes will happen on `cluster 1` because it has enough resources for scalability and utility consumption.

Specification details for creating a `blue-apd` deployment are listed below: -

1. The name of the deployment is `blue-apd`.
2. Use the label `type-one: blue`.
3. Use the image `kodekloud/webapp-color:v1`.
4. Add labels to the pod `type-one: blue` and `version: v1`.

Specification details for creating a `green-apd` deployment are listed below: -

1. The name of the deployment is `green-apd`.
2. Use the label `type-two: green`.
3. Use the image `kodekloud/webapp-color:v2`.
4. Add labels to the pod `type-two: green` and `version: v1`.

We have to create a service called `route-apd-svc` for these deployments. Details are here: -

1. The name of the service is `route-apd-svc`.
2. Use the correct service type to access the application from outside the cluster and application should listen on port `8080`.
3. Use the selector label `version: v1`.

**NOTE**: - We do not need to increase replicas for the deployments, and all the resources should be created in the `default` namespace.

You can check the status of the application from the terminal by running the `curl` command with the following syntax:

```
curl http://cluster1-controlplane:NODE-PORT
```

You can SSH into the `cluster1` using `ssh cluster1-controlplane` command.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -

```bash
kubectl config use-context cluster1
```

In this task, we will use the `kubectl` command. Here are the steps: -

1. Use the `kubectl create` command to create a deployment manifest file as follows: -

```bash
kubectl create deployment blue-apd --image=kodekloud/webapp-color:v1 --dry-run=client -o yaml > <FILE-NAME-1>.yaml
```

Do the same for the other deployment and service.

```bash
kubectl create deployment green-apd --image=kodekloud/webapp-color:v2 --dry-run=client -o yaml > <FILE-NAME-2>.yaml
```

```bash
kubectl create service nodeport route-apd-svc --tcp=8080:8080 --dry-run=client -oyaml > <FILE-NAME-3>.yaml
```

1. Open the file with any text editor such as `vi` or `nano` and make the changes as per given in the specifications. It should look like this: -

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    type-one: blue
  name: blue-apd
spec:
  replicas: 7
  selector:
    matchLabels:
      type-one: blue
      version: v1
  template:
    metadata:
      labels:
        version: v1
        type-one: blue
    spec:
      containers:
      - image: kodekloud/webapp-color:v1
        name: blue-apd
```

We will deploy a total of 10 application pods. Also, we have to route `70%` traffic to `blue-apd` and `30%` traffic to the `green-apd` deployment according to the task description.
  
Since the service distributes traffic to all pods equally, we have to set the replica count `7` to the `blue-apd` deployment so that the given service will send `~70%` traffic to the deployment pods.

`green-apd` deployment should look like this: -

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    type-two: green
  name: green-apd
spec:
  replicas: 3
  selector:
    matchLabels:
      type-two: green
      version: v1
  template:
    metadata:
      labels:
        type-two: green
        version: v1
    spec:
      containers:
      - image: kodekloud/webapp-color:v2
        name: green-apd
```

`route-apd-svc` service should look like this: -

```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: route-apd-svc
  name: route-apd-svc
spec:
  type: NodePort
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    version: v1
```

1. Now, create a deployment and service by using the `kubectl create -f` command: -

```bash
kubectl create -f <FILE-NAME-1>.yaml -f <FILE-NAME-2>.yaml -f <FILE-NAME-3>.yaml
```

</details>

**Details (Verification):**

* Is blue deployment configured correctly?
* Is green deployment configured correctly?
* Is service configured correctly?

-----

## Services And Networking

### Question 9

**Context:** `kubectl config use-context cluster3`

**Task:**
We have deployed several applications in the `ns-ckad17-svcn` namespace that are exposed inside the cluster via ClusterIP.
  
Your task is to create a LoadBalancer type service that will serve traffic to the applications based on its labels. Create the resources as follows:

* Service `lb1-ckad17-svcn` for serving traffic at `port 31890` to pods with labels `"exam=ckad, criteria=location"`.
  
* Service `lb2-ckad17-svcn` for serving traffic at `port 31891` to pods with labels `"exam=ckad, criteria=cpu-high"`.

<details>
<summary><b>View Solution</b></summary>

To create the loadbalancer for the pods with the specified lables, first we need to find the pods with the mentioned lables.

* To get pods with labels "exam=ckad, criteria=location"

```yaml
kubectl -n ns-ckad17-svcn get pod -l exam=ckad,criteria=location
-----
NAME READY STATUS RESTARTS AGE
geo-location-app 1/1 Running 0 10m
```- Similarly to get pods with labels "exam=ckad,criteria=cpu-high".
```yaml
  kubectl -n ns-ckad17-svcn get pod -l exam=ckad,criteria=cpu-high
  -----
  NAME READY STATUS RESTARTS AGE
  cpu-load-app 1/1 Running 0 11m
  ```

  Now we know which pods use the labels, we can create the LoadBalancer type service using the imperative command.

```yaml
  kubectl -n ns-ckad17-svcn expose pod geo-location-app --type=LoadBalancer --name=lb1-ckad17-svcn
  ```

  Similarly, create the another service.

```yaml
  kubectl -n ns-ckad17-svcn expose pod cpu-load-app --type=LoadBalancer --name=lb2-ckad17-svcn
  ```

- Once the services are created, you can edit the services to use the correct nodePorts as per the question using `kubectl -n ns-ckad17-svcn edit svc lb2-ckad17-svcn`.

</details>

**Details (Verification):**

* Is service `lb1-ckad17-svcn` created?
* Is service `lb2-ckad17-svcn` created?
* Are correct labels used for lb1 ?
* Are correct labels used for lb2?
* Is port `31890` configured for lb1 ?
* Is port `31891` configured for lb2?

-----

### Question 10

**Context:** `kubectl config use-context cluster3`

**Task:**
We have created a Network Policy `netpol-ckad13-svcn` that allows traffic only to specific pods and it allows traffic only from pods with specific labels.

Your task is to edit the policy so that it allows traffic from pods with labels `access = allowed`.

Do not change the existing rules in the policy.

<details>
<summary><b>View Solution</b></summary>

To edit the existing network policy use the following command:

```yaml
kubectl edit netpol netpol-ckad13-svcn
```

Edit the policy as follows:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpol-ckad13-svcn
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: kk-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: server
    #add the following in the manifest
    - podSelector:
        matchLabels:
          access: allowed
```

</details>

**Details (Verification):**

* Does network policy exist?
* Does the policy allows traffic from pods with label `tier=server`?
* Does the policy allows traffic from pods with label `access=allowed`?
* Does the policy denies the traffic from pods which don't have specified labels?

-----

### Question 11

**Context:** `kubectl config use-context cluster1`

**Task:**
We have created a deployment of an application named `nginx-app-ckad` in the `default` namespace.

Configure a service named `nginx-svcn` for the application, which exposes the pods on multiple ports with different protocols.

* Expose port 80 using TCP with the name `http`
* Expose port 443 using TCP with the name `https`

<details>
<summary><b>View Solution</b></summary>

The pod `app-ckad-svcn` is deployed in default namespace.

* To view the pod along with labels, use the following command.

```yaml
student-node ~ ➜ kubectl get deployments.apps nginx-app-ckad --show-labels
NAME READY UP-TO-DATE AVAILABLE AGE LABELS
nginx-app-ckad 3/3 3 3 5m21s app=nginx-app-ckad
```

* We will use those labels to create the service.
  
* Create a service using the following manifest. It will create a service with multiple ports expose with different protocols.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svcn
  labels:
    app: nginx-app-ckad
spec:
  selector:
    app: nginx-app-ckad
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
```

</details>

**Details (Verification):**

* Is port 80 exposed using TCP?
* Is port 443 exposed using TCP?
* Is service `nginx-svcn` created?
* Are correct labels used?

-----

## Application Environment, Configuration And Security

### Question 12

**Context:** `kubectl config use-context cluster2`

**Task:**
We have a Kubernetes namespace called `ckad12-ctm-sa-aecs`, which contains a service account and a pod. Your task is to modify the pod so that it uses the service account defined in the same namespace.
  
Additionally, you need to ensure that the pod has access to the **API credentials** associated with the service account by **enabling** the automounting feature for the credentials.

<details>
<summary><b>View Solution</b></summary>

Here we will do two things:
  
Remove `automountServiceAccountToken: false` field to enable automount of creds and specifying our service account as: `serviceAccountName: ckad12-my-custom-sa-aecs`

```yaml
student-node ~ ➜ kubectl config use-context cluster2
Switched to context "cluster2".
student-node ~ ➜ kubectl get pods -n ckad12-ctm-sa-aecs ckad12-ctm-nginx-aecs -o yaml > ckad-custom-sa.yaml
student-node ~ ➜ vim ckad-custom-sa.yaml
student-node ~ ➜ cat ckad-custom-sa.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ckad12-ctm-nginx-aecs
  namespace: ckad12-ctm-sa-aecs
spec:
  # automountServiceAccountToken: false *removed to enable automount of creds
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
  serviceAccountName: ckad12-my-custom-sa-aecs # using custom sa
student-node ~ ➜ kubectl replace -f ckad-custom-sa.yaml --force
pod "ckad12-ctm-nginx-aecs" deleted
pod/ckad12-ctm-nginx-aecs replaced
```

</details>

**Details (Verification):**

* Is the automounting feature enabled?
* Is the correct service account used?

-----

### Question 13

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a `role` named `pod-creater` in the `ckad20-auth-aecs` namespace, and grant only the **list**, **create** and **get** permissions on `pods` resources.  
  
Create a `role binding` named `mock-user-binding` in the same namespace, and assign the **pod-creater** role to a user named `mock-user`.

<details>
<summary><b>View Solution</b></summary>

```yaml
student-node ~ ➜ kubectl config use-context cluster3
Switched to context "cluster3".
student-node ~ ➜ kubectl create ns ckad20-auth-aecs
student-node ~ ➜ kubectl create role pod-creater --namespace=ckad20-auth-aecs --verb=list,create,get --resource=pods
role.rbac.authorization.k8s.io/pod-creater created
student-node ~ ➜ kubectl create rolebinding mock-user-binding --namespace=ckad20-auth-aecs --role=pod-creater --user=mock-user
rolebinding.rbac.authorization.k8s.io/mock-user-binding created
# Now let's validate if our role and role binding is working as expected
student-node ~ ➜ kubectl auth can-i create pod --as mock-user
no
student-node ~ ✖ kubectl auth can-i create pod --as mock-user --namespace ckad20-auth-aecs
yes
```

</details>

**Details (Verification):**

* Is correct resource for role "pod-creater" specified?
* Are correct "verbs" specified for the role?
* Is correct role used for rolebinding?
* Is correct user specified for rolebinding?

-----

### Question 14

**Context:** `kubectl config use-context cluster3`

**Task:**
In the `ckad14-sa-projected` namespace, configure the `ckad14-api-pod` Pod to include a **projected volume** named `vault-token`.

Mount the service account token to the container at `/var/run/secrets/tokens`, with an expiration time of `7000` seconds.
  
Additionally, set the intended audience for the token to `vault` and path to `vault-token`.

<details>
<summary><b>View Solution</b></summary>

```yaml
student-node ~ ➜ kubectl config use-context cluster3
Switched to context "cluster3".
student-node ~ ➜ k get pod -n ckad14-sa-projected ckad14-api-pod -o yaml > ckad-pro-vol.yaml
student-node ~ ➜ cat ckad-pro-vol.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ckad14-api-pod
  namespace: ckad14-sa-projected
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
    .
    .
    .
    volumeMounts: # Added
    - mountPath: /var/run/secrets/tokens # Added
      name: vault-token # Added
  .
  .
  .
  serviceAccount: ckad14-sa
  serviceAccountName: ckad14-sa
  volumes:
  - name: vault-token # Added
    projected: # Added
      sources: # Added
      - serviceAccountToken: # Added
          path: vault-token # Added
          expirationSeconds: 7000 # Added
          audience: vault # Added
student-node ~ ➜ k replace -f ckad-pro-vol.yaml --force
pod "ckad14-api-pod" deleted
pod/ckad14-api-pod replaced
```

</details>

**Details (Verification):**

* Is a "projected volume" ?
* Is volume configured correctly ?
* Is volume mounted correctly ?

-----

### Question 15

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a custom resource `my-anime` of kind `Anime` with the below specifications:

Name of Anime: `Naruto`  
Episode Count: `220`  
  
`TIP`: You may find the respective CRD with **anime** substring in it.

<details>
<summary><b>View Solution</b></summary>

```yaml
student-node ~ ➜ kubectl config use-context cluster3
Switched to context "cluster3".
student-node ~ ➜ kubectl get crd | grep -i anime
animes.animes.k8s.io
student-node ~ ➜ kubectl get crd animes.animes.k8s.io \
-o json \
| jq .spec.versions[].schema.openAPIV3Schema.properties.spec.properties
{
"animeName": {
"type": "string"
},
"episodeCount": {
"maximum": 300,
"minimum": 24,
"type": "integer"
}
}
student-node ~ ➜ k api-resources | grep anime
animes an animes.k8s.io/v1alpha1 true Anime
student-node ~ ➜ cat << YAML | kubectl apply -f -
apiVersion: animes.k8s.io/v1alpha1
kind: Anime
metadata:
  name: my-anime
spec:
  animeName: "Naruto"
  episodeCount: 220
YAML
anime.animes.k8s.io/my-anime created
student-node ~ ➜ k get an my-anime
NAME AGE
my-anime 23s
```

</details>

**Details (Verification):**

* Are correct specifications used for custom resource?

-----

### Question 16

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a service account `dashboard-sa` in the namespace `levantest`.

<details>
<summary><b>View Solution</b></summary>

We can use the below command to create the service account.  
  
`kubectl create serviceaccount dashboard-sa -n levantest`

</details>

**Details (Verification):**

* Is service account created ?

-----

## Application Observability And Maintenance

### Question 17

**Context:** `kubectl config use-context cluster1`

**Task:**
A pod named `ckad-nginx-pod-aom` is deployed and exposed with a service `ckad-nginx-service-aom`, but it seems the service is not configured properly and is not selecting the correct pod.
  
Make the required changes to service and ensure the endpoint is configured for service.

<details>
<summary><b>View Solution</b></summary>

Check for service end point by using

```bash
kubectl describe svc ckad-nginx-service-aom
```

we can see below output as below

```bash
kubectl describe svc ckad-nginx-service-aom
Name: ckad-nginx-service-aom
Namespace: default
Labels: <none>
Annotations: <none>
Selector: app=ngnix
Type: ClusterIP
IP Family Policy: SingleStack
IP Families: IPv4
IP: 10.43.134.231
IPs: 10.43.134.231
Port: 80-80 80/TCP
TargetPort: 80/TCP
Endpoints: <none>
Session Affinity: None
Events: <none>
```

we can see there **endpoint** value as **none**. Let's debug this we can see **selector as app=ngnix** . Lets check label value in pod.

```bash
kubectl get pod ckad-nginx-pod-aom -o json | jq -r .metadata.labels
"app": "nginx"
```

we can see here selector is **mis-spelled** in service. so edit service and check for endpoint value.

```bash
kubectl get ep ckad-nginx-service-aom
NAME ENDPOINTS AGE
ckad-nginx-service-aom 10.42.2.4:80,10.42.2.5:80 4m44s
```

</details>

**Details (Verification):**

* Is endpoint configured?

-----

### Question 18

**Context:** `kubectl config use-context cluster2`

**Task:**
View the metrics (CPU and Memory) of the node `cluster2-node01` and copy the output to the `/root/node-metrics` file in `nodeName, CPU and memory`.

<details>
<summary><b>View Solution</b></summary>

Use the following command to get details:

```bash
kubectl top nodes cluster2-node01 > /root/node-metrics
```

</details>

**Details (Verification):**

* Output saved to a file

-----

### Question 19

**Context:** `kubectl config use-context cluster2`

**Task:**
A template to create a Kubernetes pod is stored at `/root/goproxy.yaml` on the `student-node`. Set the `initialDelaySeconds to 5`. However, using this template results in an error.  
  
Fix the issue with this template and use it to create the pod. Once created, watch the pod for a minute or two to make sure it is stable i.e., it is not crashing or restarting.

<details>
<summary><b>View Solution</b></summary>

##### Try to apply the template

```yaml
kubectl apply -f goproxy.yaml
```

You will see error:

```yaml
error: error validating "goproxy.yaml": error validating data: [ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): unknown field "command" in io.k8s.api.core.v1.HTTPGetAction, ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): missing required field "port" in io.k8s.api.core.v1.HTTPGetAction]; if you choose to ignore these errors, turn validation off with --validate=false
```

From the error you can see that the error is for liveness probe, so let's open the template to find out:

```bash
vi goproxy.yaml
```

Under `livenessProbe:` you will see the type is `httpGet` however the rest of the options are command based so this probe should be of `exec` type.

* Change `httpGet` to `exec`

##### Try to apply the template now

```yaml
kubectl apply -f goproxy.yaml
```

Cool it worked, now let's watch the POD status, after few seconds you will notice that POD is restarting. So let's check the logs/events

```bash
kubectl get event --field-selector involvedObject.name=goproxy
```

You will see an error like:

```yaml
21s Warning Unhealthy pod/goproxy Liveness probe failed: cat: can't open '/healthcheck': No such file or directory
```

So seems like Liveness probe is failing, lets look into it:

```bash
vi goproxy.yaml
```

Notice the command `- sleep 3 ; touch /healthcheck; sleep 30;sleep 30000` it starts with a delay of `3` seconds, but the liveness probe `initialDelaySeconds` is set to `1` and `failureThreshold` is also `1`. Which means the POD will fail just after first attempt of liveness check which will happen just after `1` second of pod start. So to make it stable we must increase the `initialDelaySeconds` to at least `5`

```bash
vi goproxy.yaml
```

* Change `initialDelaySeconds` from `1` to `5` and save apply the changes.

Delete old pod:

```bash
kubectl delete pod goproxy
```

Apply changes:

```bash
kubectl apply -f goproxy.yaml
```

</details>

**Details (Verification):**

* Probe type changed to exec ?
* Is POD stable?
* Initial delay seconds is set to 5 ?

-----

## Service Networking

### Question 20

**Context:** `kubectl config use-context cluster3`

**Task:**
**Part I**:  
  
Create a `ClusterIP` service .i.e. `service-3421-svcn` in the `spectra-1267` ns which should expose the pods namely `pod-23` and `pod-21` with port set to `8080` and targetport to `80`.  
  
**Part II**:  
  
Store the **pod names** and their **ip addresses** from the `spectra-1267` ns at `/root/pod_ips_cka05_svcn` where the output is sorted by their IP's.

Please ensure the format as shown below:

```
POD_NAME        IP_ADDR
pod-1           ip-1
pod-3           ip-2
pod-2           ip-3
...
```

<details>
<summary><b>View Solution</b></summary>

Switching to `cluster3`:

```yaml
kubectl config use-context cluster3
```

The easiest way to route traffic to a specific pod is by the use of `labels` and `selectors` . List the pods along with their labels:

```yaml
student-node ~ ➜ kubectl get pods --show-labels -n spectra-1267
NAME READY STATUS RESTARTS AGE LABELS
pod-12 1/1 Running 0 5m21s env=dev,mode=standard,type=external
pod-34 1/1 Running 0 5m20s env=dev,mode=standard,type=internal
pod-43 1/1 Running 0 5m20s env=prod,mode=exam,type=internal
pod-23 1/1 Running 0 5m21s env=dev,mode=exam,type=external
pod-32 1/1 Running 0 5m20s env=prod,mode=standard,type=internal
pod-21 1/1 Running 0 5m20s env=prod,mode=exam,type=external
```

Looks like there are a lot of pods created to confuse us. But we are only concerned with the labels of `pod-23` and `pod-21`.
  
As we can see both the required pods have labels `mode=exam,type=external` in common. Let's confirm that using kubectl too:

```yaml
student-node ~ ➜ kubectl get pod -l mode=exam,type=external -n spectra-1267
NAME READY STATUS RESTARTS AGE
pod-23 1/1 Running 0 9m18s
pod-21 1/1 Running 0 9m17s
```

Nice!! Now as we have figured out the labels, we can proceed further with the creation of the service:

```yaml
student-node ~ ➜ kubectl create service clusterip service-3421-svcn -n spectra-1267 --tcp=8080:80 --dry-run=client -o yaml > service-3421-svcn.yaml
```

Now modify the service definition with selectors as required before applying to k8s cluster:

```yaml
student-node ~ ➜ cat service-3421-svcn.yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: service-3421-svcn
  name: service-3421-svcn
  namespace: spectra-1267
spec:
  ports:
  - name: 8080-80
    port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: service-3421-svcn # delete
    mode: exam # add
    type: external # add
  type: ClusterIP
status:
  loadBalancer: {}
```

Finally let's apply the service definition:

```yaml
student-node ~ ➜ kubectl apply -f service-3421-svcn.yaml
service/service-3421 created
student-node ~ ➜ k get ep service-3421-svcn -n spectra-1267
NAME ENDPOINTS AGE
service-3421 10.42.0.15:80,10.42.0.17:80 52s
```

To store all the pod name along with their IP's , we could use imperative command as shown below:

```yaml
student-node ~ ➜ kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP
POD_NAME IP_ADDR
pod-12 10.42.0.18
pod-23 10.42.0.19
pod-34 10.42.0.20
pod-21 10.42.0.21
...
# store the output to /root/pod_ips
student-node ~ ➜ kubectl get pods -n spectra-1267 -o=custom-columns='POD_NAME:metadata.name,IP_ADDR:status.podIP' --sort-by=.status.podIP > /root/pod_ips_cka05_svcn
```

</details>

**Details (Verification):**

* "service-3421-svcn" exists?
* service-3421-svcn is of "type: ClusterIP"?
* port: 8080?
* targetPort: 80?
* Service only exposes pod "pod-23" and "pod-21"?
* correct and sorted output stored in "/root/pod_ips_cka05_svcn"?

-----
