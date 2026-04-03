# Mock Exam 1

## Application Design and Build

### Question 1

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-multi-containers` namespace, create a pod named `tres-containers-pod`, which has 3 containers matching the below requirements:

* The first container named `primero` runs `busybox:1.28` image and has `ORDER=FIRST` environment variable.
* The second container named `segundo` runs `nginx:1.17` image and is exposed at port `8080`.
* The last container named `tercero` runs `busybox:1.31.1` image and has `ORDER=THIRD` environment variable.

**NOTE:** All pod containers should be in the running state.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create desired pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: primero
  name: tres-containers-pod
  namespace: ckad-multi-containers
spec:
  containers:
  - env:
    - name: ORDER
      value: FIRST
    image: busybox:1.28
    name: primero
    command:
    - /bin/sh
    - -c
    - sleep 3600;
    resources: {}
  - image: nginx:1.17
    name: segundo
    ports:
    - containerPort: 8080
    resources: {}
  - env:
    - name: ORDER
      value: THIRD
    image: busybox:1.31.1
    name: tercero
    command:
    - /bin/sh
    - -c
    - sleep 3600;
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</details>

**Details (Verification):**

* Is the `primero` container running?
* Is the `primero` container use `busybox:1.28` image?
* Is the `primero` container environment variable well configured?
* Does the `segundo` container running?
* Does the `segundo` container use `nginx:1.17` image?
* Is the `segundo` container expose at port `8080`?
* Is the `tercero` container running?
* Is the `tercero` container use `busybox:1.31.1` image?
* Is the `tercero` container environment variable well configured?

---

### Question 2

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a storage class with the name `banana-sc-ckad08-str` as per the properties given below:

* Provisioner should be `kubernetes.io/no-provisioner`,
* Volume binding mode should be `WaitForFirstConsumer`.
* Volume expansion should be enabled.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create desired storage-class:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: banana-sc-ckad08-str
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

</details>

**Details (Verification):**

* Is `banana-sc-ckad08-str` storage class created?

---

### Question 3

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad-job` namespace, create a cronjob named `simple-node-job` to run every 30 minutes to list all the running processes inside a container that used `node` image (the command needs to be run in a shell).

In Unix-based operating systems, `ps -eaf` can be use to list all the running processes.

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-node-job
  namespace: ckad-job
spec:
  schedule: "*/30 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: simple-node-job
            image: node
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - ps -eaf
          restartPolicy: OnFailure
```

Then use `kubectl apply -f file_name.yaml` to create the required object.

</details>

**Details (Verification):**

* Is cronjob `simple-node-job` created?
* Is the container image `node`?
* Does cronjob run `ps -eaf` command?
* Does cronjob run every 30 minutes?

---

### Question 4

**Context:** `kubectl config use-context cluster3`

**Task:**
In the `ckad-pod-design` namespace, create a pod called `ckad-ubuntu-qwfefefwe` that runs a `ubuntu` image.

The pod's container should be named `ubuntu-server`; the container will sleep for `3600` seconds.

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ckad-ubuntu-qwfefefwe
  name: ckad-ubuntu-qwfefefwe
  namespace: ckad-pod-design
spec:
  containers:
  - command:
    - sleep
    - "3600"
    image: ubuntu
    name: ubuntu-server
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.

Alternatively, you can use this command for similar outcome:

```bash
kubectl run ckad-ubuntu-qwfefefwe \
  --image=ubuntu \
  --restart=Always \
  --namespace=ckad-pod-design \
  --overrides='
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "ckad-ubuntu-qwfefefwe"
  },
  "spec": {
    "restartPolicy": "Always",
    "containers": [
      {
        "name": "ubuntu-server",
        "image": "ubuntu",
        "command": ["sleep", "3600"]
      }
    ]
  }
}'
```

</details>

**Details (Verification):**

* Is the pod `ckad-ubuntu-qwfefefwe` running?
* Is the container image `ubuntu`?
* Is the required command available?

---

## Application Deployment

### Question 5

**Context:** `kubectl config use-context cluster3`

**Task:**
In this task, we have to create two identical environments that are running different versions of the application. The team decided to use the Blue/green deployment method to deploy a total of 10 application pods which can mitigate common risks such as downtime and rollback capability.

Also, we have to route traffic in such a way that `30%` of the traffic is sent to the `green-apd` environment and the rest is sent to the `blue-apd` environment. All the development processes will happen on `cluster 3` because it has enough resources for scalability and utility consumption.

Specification details for creating a `blue-apd` deployment are listed below: -

* The name of the deployment is `blue-apd`.
* Use the label `type-one: blue`.
* Use the image `kodekloud/webapp-color:v1`.
* Add labels to the pod `type-one: blue` and `version: v1`.

Specification details for creating a `green-apd` deployment are listed below: -

* The name of the deployment is `green-apd`.
* Use the label `type-two: green`.
* Use the image `kodekloud/webapp-color:v2`.
* Add labels to the pod `type-two: green` and `version: v1`.

We have to create a service called `route-apd-svc` for these deployments. Details are here: -

* The name of the service is `route-apd-svc`.
* Use the correct service type to access the application from outside the cluster and application should listen on port `8080`.
* Use the selector label `version: v1`.

NOTE: - We do not need to increase replicas for the deployments, and all the resources should be created in the `default` namespace.

You can check the status of the application from the terminal by running the `curl` command with the following syntax:
`curl http://cluster3-controlplane:NODE-PORT`

You can SSH into the `cluster3` using `ssh cluster3-controlplane` command.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
`kubectl config use-context cluster3`

In this task, we will use the `kubectl` command. Here are the steps: -

Use the `kubectl create` command to create a deployment manifest file as follows: -
`kubectl create deployment blue-apd --image=kodekloud/webapp-color:v1 --dry-run=client -o yaml > <FILE-NAME-1>.yaml`

Do the same for the other deployment and service.
`kubectl create deployment green-apd --image=kodekloud/webapp-color:v2 --dry-run=client -o yaml > <FILE-NAME-2>.yaml`

`kubectl create service nodeport route-apd-svc --tcp=8080:8080 --dry-run=client -oyaml > <FILE-NAME-3>.yaml`

Open the file with any text editor such as `vi` or `nano` and make the changes as per given in the specifications. It should look like this: -

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

Now, create a deployment and service by using the `kubectl create -f` command: -
`kubectl create -f <FILE-NAME-1>.yaml -f <FILE-NAME-2>.yaml -f <FILE-NAME-3>.yaml`

</details>

**Details (Verification):**

* Is blue deployment configured correctly?
* Is green deployment configured correctly?
* Is service configured correctly?

---

### Question 6

**Context:** `kubectl config use-context cluster3`

**Task:**
On cluster3, in the `dev-001` namespace, one of the interns deployed one web application called `news-apd`.
After successfully deploying on the worker node, we start getting alerts about the pod crashing.

We want you to inspect the `dev-001` namespace and fix those issues.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
`kubectl config use-context cluster3`

In this task, we will use the `kubectl get`, `kubectl describe`, `kubectl logs` and `kubectl edit` commands. Here are the steps: -

To check all the resources in the specific namespaces in the `cluster3`, we would have to run the following command:
`kubectl get all -n dev-001`

It will list all the available resources of the `dev-001` namespace.

We can see that one of the pods is in an error state. Use the `kubectl describe` command to get detailed information of that pod: -
`kubectl describe -n dev-001 po <POD-NAME>`

The output of the `kubectl describe` command includes information about the Pod's status, including its IP address, the containers running in the Pod, and their current state.
It also includes information about the Pod's configuration, such as its labels, annotations, and resource requirements.

Now, we will use the `kubectl logs` command to retrieve the logs of a container running in a Pod. Use the following command as follows: -
`kubectl logs -n dev-001 <POD-NAME> <CONTAINER-NAME>`

Here, `<POD-NAME>` is the name of the Pod in which the container is running, and `<CONTAINER-NAME>` is the name of the container whose logs we want to retrieve. If the Pod has only one container, we can omit the `<CONTAINER-NAME>` argument.

In the logs, we will see that there is a typo in the `sleep` command which needs to be correct. Use the `kubectl edit` command as follows: -
`kubectl edit deploy -n dev-001 news-apd`

After fixing the typo, press the `ESC` button and type `:wq`. This command will save the changes to the file and then quit `vi` editor.

Run the `kubectl get` command again to check the pod's status.
`kubectl get pods -n dev-001`

The pod should be running.

</details>

**Details (Verification):**

* Is deployment running?

---

### Question 7

**Context:** `kubectl config use-context cluster2`

**Task:**
On the `student-node`, a Helm chart repository is given under the `/opt/` path. It contains the files that describe a set of Kubernetes resources that can be deployed as a single unit. The files have some issues. Fix those issues and deploy them with the following specifications: -

* The release name should be `webapp-color-apd`.
* All the resources should be deployed on the `frontend-apd` namespace.
* The service type should be `node port`.
* Scale the deployment to `3`.
* Application version should be `1.20.0`.

NOTE: - Remember to make necessary changes in the `values.yaml` and `Chart.yaml` files according to the specifications, and, to fix the issues, inspect the template files.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
`kubectl config use-context cluster2`

In this task, we will use the `helm` commands. Here are the steps: -

First, check the given namespace; if it doesn't exist, we must create it first; otherwise, it will give an error `"namespaces not found"` while installing the helm chart.
To check all the namespaces in the `cluster2`, we would have to run the following command: -
`kubectl get ns`

It will list all the namespaces. If the given namespace doesn't exist, then run the following command: -
`kubectl create ns frontend-apd`

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

But in our case, there are some issues with the given templates.

* Deployment apiVersion needs to be correctly written. It should be `apiVersion: apps/v1`.
* In the service YAML, there is a typo in the template variable `{{ .Values.service.name }}` because of that, it's not able to reference the value of the name field defined in the `values.yaml` file for the Kubernetes service that is being created or updated.

Now run the following command to install the helm chart in the `frontend-apd` namespace: -
`helm install webapp-color-apd -n frontend-apd ./webapp-color-apd`

Use the `helm ls` command to list the release deployed using helm.
`helm ls -n frontend-apd`

</details>

**Details (Verification):**

* Is the release name set?
* Are the resources deployed on given namespace?
* Is the service type defined?
* Is the deployment scaled?
* Is the application version set?
* Are resources running?

---

## Services and Networking

### Question 8

**Context:** `kubectl config use-context cluster3`

**Task:**
We have deployed a pod `pod22-ckad-svcn` in the default namespace. Create a service `svc22-ckad-svcn` that will expose the pod at port `6335`.

Note: Use the imperative command for the above scenario.

<details>
<summary><b>View Solution</b></summary>

To use the cluster 3, switch the context using:
`kubectl config use-context cluster3`

To expose the pod `pod22-ckad-svcn` at port 6335, we can use the following imperative command.
`kubectl expose pod pod22-ckad-svcn --name=svc22-ckad-svcn --port=6335`

It will create a service with name `svc22-ckad-svcn` and pod will be exposed at port 6335.

</details>

**Details (Verification):**

* Is service `svc22-ckad-svcn` created?
* Is the pod exposed at port `6335`?

---

### Question 9

**Context:** `kubectl config use-context cluster3`

**Task:**
You are requested to create a network policy named `deny-all-svcn` that denies all incoming and outgoing traffic to `ckad12-svcn` namespace.

Note: The namespace `ckad12-svcn` doesn't exist. Create the namespace before creating the Policy.

<details>
<summary><b>View Solution</b></summary>

Create the namespace using the following command - `kubectl create ns ckad12-svcn`

Create the network policy in the namespace `ckad12-svcn` using the following manifest.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-svcn
  namespace: ckad12-svcn
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

</details>

**Details (Verification):**

* Is namespace created?
* Is network Policy created?
* Does it restricts all Ingress traffic?
* Does it restricts all Egress connectivity?

---

### Question 10

**Context:** `kubectl config use-context cluster3`

**Task:**
We have an **external** webserver running on `student-node` which is exposed at port `9999`.

We have also created a service called `external-webserver-ckad01-svcn` that can connect to our local webserver from within the `cluster3` but, at the moment, it is not working as expected.

Fix the issue so that other pods within `cluster3` can use `external-webserver-ckad01-svcn` service to access the webserver.

<details>
<summary><b>View Solution</b></summary>

Let's check if the webserver is working or not:

```bash
student-node ~ ➜  curl student-node:9999
...<h1>Welcome to nginx!</h1>
...
```

Now we will check if service is correctly defined:

```bash
student-node ~ ➜  kubectl describe svc external-webserver-ckad01-svcn 
Name:              external-webserver-ckad01-svcn
Namespace:         default
.
.
Endpoints:         <none> # there are no endpoints for the service
...
```

As we can see there is no endpoints specified for the service, hence we won't be able to get any output. Since we can not destroy any k8s object, let's create the endpoint manually for this service as shown below:

```bash
student-node ~ ➜  export IP_ADDR=$(ifconfig eth0 | grep 'inet ' | awk '{print $2}')

student-node ~ ➜ kubectl apply -f - <<EOF
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-webserver-ckad01-svcn
  labels:
    kubernetes.io/service-name: external-webserver-ckad01-svcn
addressType: IPv4
ports:
  - protocol: TCP
    port: 9999
endpoints:
  - addresses:
      - $IP_ADDR
EOF
```

Finally check if the `curl test` works now:

```bash
student-node ~ ➜  kubectl --context cluster3 run --rm  -i test-curl-pod --image=curlimages/curl --restart=Never -- curl -m 2 external-webserver-ckad01-svcn...
<title>Welcome to nginx!</title>...
```

</details>

**Details (Verification):**

* Do endpoints exist for `external-webserver-ckad01-svcn` service ?
* Is correct port specified ?
* Can other pods use `external-webserver-ckad01-svcn` service to access webserver?

---

### Question 11

**Context:** `kubectl config use-context cluster2`

**Task:**
Deploy a pod with name `webapp-svcn` using the `kodekloud/webapp-color` image with the label `tier=msg`.

Now, Create a service `webapp-service-svcn` to expose the pod `webapp-svcn` application within the cluster on port `6379`.

<details>
<summary><b>View Solution</b></summary>

Switch to `cluster2`:
`kubectl config use-context cluster2`

On student-node, use the command:
`kubectl run webapp-svcn --image=kodekloud/webapp-color -l tier=msg`

Now run the command:
`kubectl expose pod webapp-svcn --port=6379 --name webapp-service-svcn`

</details>

**Details (Verification):**

* Pod Name: `webapp-svcn`.
* Image used is `kodekloud/webapp-color`.
* Pod is created with label `tier=msg`.
* Is service `webapp-service-svcn` created?
* Is port `6379` configured for service?
* Is service created is of type "ClusterIP"?
* Does the service created use the right labels?

---

### Question 12

**Context:** `kubectl config use-context cluster1`

**Task:**
For this scenario, create a Service called `ckad12-service` that routes traffic to an external IP address.

Please note that service should listen on `port 53` and be of type `ExternalName`. Use the external IP address 8.8.8.8

Create the service in the `default` namespace.

<details>
<summary><b>View Solution</b></summary>

Create the service using the following manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ckad12-service
spec:
  type: ExternalName
  externalName: 8.8.8.8
  ports:
    - name: http
      port: 53
      targetPort: 53
```

</details>

**Details (Verification):**

* Is service `ckad12-service` created?
* Is service of type - "ExternalName"?
* Is service configured with externalName - `8.8.8.8`?

---

## Application Environment, Configuration and Security

### Question 13

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a pod named `ckad17-qos-aecs-3` in namespace `ckad17-nqoss-aecs` with image `nginx` and container name `ckad17-qos-ctr-3-aecs`.

Define other fields such that the Pod is configured to use the Quality of Service (QoS) class of `Burstable`.

Also retrieve the `name` and `QoS` class of each Pod in the namespace `ckad17-nqoss-aecs` in the below format and save the output to a file named `qos_status_aecs` in the `/root` directory.

Format:

```
NAME    QOS
pod-1   qos_class
pod-2   qos_class
```

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜ kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ckad17-qos-aecs-3
  namespace: ckad17-nqoss-aecs
spec:
  containers:
  - name: ckad17-qos-ctr-3-aecs
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
EOF

pod/ckad17-qos-aecs-3 created

student-node ~ ➜  kubectl --namespace=ckad17-nqoss-aecs get pod --output=custom-columns="NAME:.metadata.name,QOS:.status.qosClass"
NAME                QOS
ckad17-qos-aecs-1   BestEffort
ckad17-qos-aecs-2   Guaranteed
ckad17-qos-aecs-3   Burstable

student-node ~ ➜  kubectl --namespace=ckad17-nqoss-aecs get pod --output=custom-columns="NAME:.metadata.name,QOS:.status.qosClass" > /root/qos_status_aecs
```

</details>

**Details (Verification):**

* Is the pod created with the "QoS" class of 'Burstable'?
* Is the QoS class of each pod retrieved in the format mentioned?

---

### Question 14

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a custom resource `my-anime` of kind `Anime` with the below specifications:

* Name of Anime: `Death Note`
* Episode Count: `37`

TIP: You may find the respective CRD with `anime` substring in it.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  kubectl get crd | grep -i anime
animes.animes.k8s.io

student-node ~ ➜  kubectl get crd animes.animes.k8s.io \
                 -o json \
                 | jq .spec.versions[].schema.openAPIV3Schema.properties.spec.properties
{
  "animeName": {
    "type": "string"
  },
  "episodeCount": {
    "maximum": 52,
    "minimum": 24,
    "type": "integer"
  }
}

student-node ~ ➜  k api-resources | grep anime
animes                            an           animes.k8s.io/v1alpha1                 true         Anime

student-node ~ ➜  cat << YAML | kubectl apply -f -
 apiVersion: animes.k8s.io/v1alpha1
 kind: Anime
 metadata:
   name: my-anime
 spec:
   animeName: "Death Note"
   episodeCount: 37
YAML
anime.animes.k8s.io/my-anime created

student-node ~ ➜  k get an my-anime 
NAME       AGE
my-anime   23s
```

</details>

**Details (Verification):**

* Are correct specifications used for custom resource?

---

### Question 15

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a ConfigMap named `ckad04-config-multi-env-files-aecs` in the `default` namespace from the environment(env) files provided at `/root/ckad04-multi-cm` directory.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl create configmap ckad04-config-multi-env-files-aecs \
         --from-env-file=/root/ckad04-multi-cm/file1.properties \
         --from-env-file=/root/ckad04-multi-cm/file2.properties
configmap/ckad04-config-multi-env-files-aecs created

student-node ~ ➜  k get cm ckad04-config-multi-env-files-aecs -o yaml
```

```yaml
apiVersion: v1
data:
  allowed: "true"
  difficulty: fairlyEasy
  exam: ckad
  modetype: openbook
  practice: must
  retries: "2"
kind: ConfigMap
metadata:
  name: ckad04-config-multi-env-files-aecs
  namespace: default
```

</details>

**Details (Verification):**

* Is ConfigMap created with proper configuration ?

---

### Question 16

**Context:** `kubectl config use-context cluster3`

**Task:**
We have already deployed the required pods and services in the namespace `ckad01-db-sec`.

Create a new secret named `ckad01-db-scrt-aecs` with the data given below.

* Secret Name: `ckad01-db-scrt-aecs`
* Secret 1: `DB_Host=sql01`
* Secret 2: `DB_User=root`
* Secret 3: `DB_Password=password123`

Configure `ckad01-mysql-server` to load environment variables from the newly created secret, where the `keys` from the secret should become the `environment variable name` in the Pod.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster3
Switched to context "cluster3".

student-node ~ ➜  k get all -n ckad01-db-sec
NAME                         READY   STATUS    RESTARTS   AGE
pod/ckad01-mysql-server   1/1     Running   0          3m13s
pod/ckad01-db-pod-aecs       1/1     Running   0          3m13s

NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/ckad01-webapp-service-aecs   NodePort    10.43.190.89    <none>        8080:30080/TCP   3m13s
service/ckad01-db-svc-aecs           ClusterIP   10.43.117.255   <none>        3306/TCP         3m13s

student-node ~ ➜  kubectl create secret generic ckad01-db-scrt-aecs \
   --namespace=ckad01-db-sec \
   --from-literal=DB_Host=sql01 \
   --from-literal=DB_User=root \
   --from-literal=DB_Password=password123
secret/ckad01-db-scrt-aecs created

student-node ~ ➜  k get -n ckad01-db-sec pod ckad01-mysql-server -o yaml > webapp-pod-sec-cfg.yaml

student-node ~ ➜  vim webapp-pod-sec-cfg.yaml

student-node ~ ➜  cat webapp-pod-sec-cfg.yaml 
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: ckad01-mysql-server
  name: ckad01-mysql-server
  namespace: ckad01-db-sec
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: ckad01-db-scrt-aecs
```

```bash
student-node ~ ➜  kubectl replace -f webapp-pod-sec-cfg.yaml --force 
pod "ckad01-mysql-server" deleted
pod/ckad01-mysql-server replaced

student-node ~ ➜  kubectl exec -n ckad01-db-sec ckad01-mysql-server -- printenv | egrep -w 'DB_Password=password123|DB_User=root|DB_Host=sql01'
DB_Password=password123
DB_User=root
DB_Host=sql01
```

</details>

**Details (Verification):**

* Is the secret created with correct values ?
* Is the pod configured to use secret as env vars?

---

### Question 17

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a `ResourceQuota` called `ckad16-rqc` in the namespace `ckad16-rqc-ns` and enforce a limit of `one` `ResourceQuota` for the namespace.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  kubectl create namespace ckad16-rqc-ns
namespace/ckad16-rqc-ns created

student-node ~ ➜  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ckad16-rqc
  namespace: ckad16-rqc-ns
spec:
  hard:
    resourcequotas: "1"
EOF

resourcequota/ckad16-rqc created

student-node ~ ➜  k get resourcequotas -n ckad16-rqc-ns
NAME              AGE   REQUEST               LIMIT
ckad16-rqc   20s   resourcequotas: 1/1   
```

</details>

**Details (Verification):**

* Is resource quota created?

---

### Question 18

**Context:** `kubectl config use-context cluster2`

**Task:**
Using the pod template on `student-node` at `/root/ckad08-dotfile-aecs.yaml` , create a pod `ckad18-secret-pod` in the namespace `ckad18-secret` with the specifications as defined below:

Define a volume section named `secret-volume` that is backed by a Kubernetes Secret named `ckad18-secret-aecs`.

Mount the `secret-volume` volume to the container's `/etc/secret-volume` directory in `read-only` mode, so that the container can access the secrets stored in the `ckad18-secret-aecs` secret.

<details>
<summary><b>View Solution</b></summary>

```yaml
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ckad18-secret-pod
  namespace: ckad18-secret
spec:
  restartPolicy: Never
  volumes:
  - name: secret-volume
    secret:
      secretName: ckad18-secret-aecs
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

* Is "read-only" volume created using "ckad18-secret-aecs" ?
* Is 'readonly' volumeMount used with correct path ?

---

## Application Observability and Maintenance

### Question 19

**Context:** `kubectl config use-context cluster1`

**Task:**
Update the newly created pod `simple-webapp-aom` with a `readinessProbe` using the given specifications.
Configure an `HTTP` readiness probe with:

* path value set to `/ready`
* port number to access container is `8080`
* `initialDelaySeconds` set to `15` (to allow app startup time)

Note: You need to recreate the pod to add the readiness probe configuration.

<details>
<summary><b>View Solution</b></summary>

Use the following YAML file and create a file - for example, `simple-webapp-aom.yaml`:

```yaml
cat <<EOF > simple-webapp-aom.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-aom
  labels:
    name: simple-webapp-aom
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-delayed-start
    ports:
    - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 5
EOF
```

To recreate the pod, run the command:
`kubectl replace -f simple-webapp-aom.yaml --force`

</details>

**Details (Verification):**

* Is pod named `simple-webapp-aom` created?
* Image Name: `kodekloud/webapp-delayed-start`
* Readiness Probe: `httpGet`
* Http Probe: `/ready`
* Http Port: `8080`

---

### Question 20

**Context:** `kubectl config use-context cluster1`

**Task:**
Pod manifest file is already given under the `/root/` directory called `ckad-pod-busybox.yaml`.

There is error with manifest file correct the file and create resource.

<details>
<summary><b>View Solution</b></summary>

You will see following error

```bash
student-node ~ ➜  kubectl create -f ckad-pod-busybox.yaml
Error from server (BadRequest): error when creating "ckad-pod-busybox.yaml": Pod in version "v1" cannot be handled as a Pod.
```

Use the following yaml file and create resource

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ckad-pod-busybox
spec:
  containers:
    - command:
        - sleep
        - "3600"
      image: busybox
      name: pods-simple-container
```

</details>

**Details (Verification):**

* Is the pod ckad-pod-busybox running?
* Is correct apiVersion used?
* Is image busybox?

---

### Question 21

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a new pod with image `redis` and name `ckad-probe` and configure the pod with `livenessProbe` with `command` `ls` and set `initialDelaySeconds` to 5 .

TIP: - Make use of the imperative command to create the above pod.

<details>
<summary><b>View Solution</b></summary>

Using imperative command

`kubectl run ckad-probe --image=redis  --dry-run=client -o yaml > ckad-probe.yaml`

Use the following YAML file update yaml with livenessProbe:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: ckad-probe
spec:
  containers:
    - image: redis
      imagePullPolicy: IfNotPresent
      name: redis
      resources: {}
      livenessProbe:
        exec:
          command:
            - ls
        initialDelaySeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

To recreate the pod, run the command:
`kubectl create -f ckad-probe.yaml`

</details>

**Details (Verification):**

* Podname: `ckad-probe`
* `redis`
* `livenessProbe`
* Command set to `ls`

---
