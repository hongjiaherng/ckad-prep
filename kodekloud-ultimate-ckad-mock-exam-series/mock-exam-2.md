# Lab: CKAD Mock Exam 2

## Application Design and Build

### Question 1

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-multi-containers` namespaces, create a `ckad-neighbor-pod` pod that matches the following requirements.

* Pod has an `emptyDir` volume named `my-vol`.
* The first container named `main-container`, runs `nginx:1.16` image. This container mounts the `my-vol` volume at `/usr/share/nginx/html` path.
* The second container is a co-located container named `neighbor-container`, and runs the `busybox:1.28` image. This container mounts the volume `my-vol` at `/var/log` path.
* Every 5 seconds, this container should write the current date along with greeting message `Hi I am from neighbor container` to `index.html` in the `my-vol` volume.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create the desired pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: ckad-multi-containers
  name: ckad-neighbor-pod
spec:
  containers:
    - image: nginx:1.16
      name: main-container
      resources: {}
      # ports:
      #   - containerPort: 80
      volumeMounts:
        - name: my-vol
          mountPath: /usr/share/nginx/html
    - image: busybox:1.28
      command:
        - /bin/sh
        - -c
        - while true; do echo $(date -u) Hi I am from neighbor container >> /var/log/index.html; sleep 5;done
      name: neighbor-container
      resources: {}
      volumeMounts:
        - name: my-vol
          mountPath: /var/log
  dnsPolicy: Default
  volumes:
    - name: my-vol
      emptyDir: {}
```

You can verify the logs with below command:

```bash
kubectl exec -n ckad-multi-containers ckad-neighbor-pod --container  main-container -- cat /usr/share/nginx/html/index.html
```

Similar logs should be displayed as below:

```text
Mon Mar 20 06:02:07 UTC 2023 Hi I am from neighbor container
Mon Mar 20 06:02:12 UTC 2023 Hi I am from neighbor container
...
```

</details>

**Details (Verification):**

* Is the main container running?
* Is the pod volume well configured?
* Is the correct message logged to main-container?

-----

### Question 2

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-multi-containers` namespace, create pod named `dos-containers-pod` which has 2 containers matching the below requirements:

* The first container named `alpha` runs the `nginx:1.17` image and has the `ROLE=SERVER` ENV variable configured.
* The second container named `beta`, runs `busybox:1.28` image. This container will print message `Hello multi-containers` (command needs to be run in shell).

**NOTE:** all containers should be in a running state to pass the validation.

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: dos-containers-pod
  name: dos-containers-pod
  namespace: ckad-multi-containers
spec:
  containers:
  - image: nginx:1.17
    env:
    - name: ROLE
      value: SERVER
    name: alpha
    resources: {}
  - command:
    - /bin/sh
    - -c
    - echo Hello multi-containers; sleep 4000;
    image: busybox:1.28
    name: beta
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.
In the exam, you can take advantage of dry-run flag to generate the YAML and modify it as question requirements.

```bash
kubectl run dos-containers-pod -n ckad-multi-containers --image=nginx:1.17 --env ROLE=SERVER --dry-run=client -o yaml > file_name.yaml
```

Next, use `vim` to modify the created YAML file, follow the question requirements.

```bash
vi file_name.yaml
```

</details>

**Details (Verification):**

* Is the first pod's container running?
* Is the first container named `alpha`?
* Is the first container run `nginx:1.17` image?
* Does the first container ENV configured correctly?
* Is the second pod's container running?
* Is the second container named `beta`?
* Does beta container run `busybox:1.28` image?
* Does `beta` container run the required command?

-----

### Question 3

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad-job` namespace, create a job called `alpine-job` that runs `top` command inside the container; use `alpine` image for this task.

<details>
<summary><b>View Solution</b></summary>

Create a YAML file with the content as below:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: alpine-job
  namespace: ckad-job
spec:
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - top
        image: alpine
        name: alpine-job
        resources: {}
      restartPolicy: Never
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.
Alternatively, you can use this command for similar outcome:

```bash
kubectl create job alpine-job --image=alpine -n ckad-job -- top
```

</details>

**Details (Verification):**

* Is the job `alpine-job` created?
* Is the container image `alpine`?
* Does the job run the `top` command?

-----

### Question 4

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad-pod-design` namespace, start a `ckad-redis-wiipwlznjy` pod running the `redis` image; the container should be named `redis-custom-annotation`.

Configure a custom `annotation` to that pod as below:

```text
KKE: https://engineer.kodekloud.com/
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
    run: ckad-redis-wiipwlznjy
  name: ckad-redis-wiipwlznjy
  namespace: ckad-pod-design
  annotations: 
    KKE: https://engineer.kodekloud.com/
spec:
  containers:
  - image: redis
    name: redis-custom-annotation
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.
Alternatively, you can use this command for similar outcome:

```bash
kubectl run ckad-redis-wiipwlznjy -n ckad-pod-design --image=redis --annotations KKE=https://kodekloud.com
```

</details>

**Details (Verification):**

* Is pod `ckad-redis-wiipwlznjy` running?
* Is the pod's container named `redis-custom-annotation`?
* Is the pod's annotation configured correctly?

-----

## Application Deployment

### Question 5

**Context:** `kubectl config use-context cluster2`

**Task:**
On cluster2, we are getting alerts for the resource consumption for the `ultra-apd` namespace. A limit is set for the `ultra-apd` namespace; if the limit is crossed, we will get the alerts.

After inspecting the namespace, the team found that the deployment scaled to 8, which is wrong.
The team wants you to decrease the deployment to 4.

<details>
<summary><b>View Solution</b></summary>

In this task, we will use the `kubectl` command. Here are the steps:

1. Use the `kubectl get` command to list the pods and deployment on the `ultra-apd` namespace.

<!-- end list -->

```bash
kubectl get pods,deployment -n ultra-apd
```

Here `-n` option stands for `namespace`, which is used to specify the namespace.

1. Inspect the deployed resources and count the number of pods.

2. Now, scale down the deployment to `4` as follows:

<!-- end list -->

```bash
kubectl scale deployment ultra-deploy-apd -n ultra-apd --replicas=4
```

The above command will scale down the number of replicas in a deployment.

</details>

**Details (Verification):**

* Is deployment scaled down?

-----

### Question 6

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a new deployment called `ocean-apd` in the default namespace using the image `kodekloud/webapp-color:v1`.
Use the following specs for the deployment:

1. Replica count should be `2`.
2. Set the Max Unavailable to `45%` and Max Surge to `55%`.
3. Create the deployment and ensure all the pods are ready.
4. After successful deployment, upgrade the deployment image to `kodekloud/webapp-color:v2` and inspect the deployment rollout status.
5. Check the rolling history of the deployment and on the `student-node`, save the `current` revision count number to the `/opt/ocean-revision-count.txt` file.
6. Finally, perform a rollback and revert the deployment image to the older version.

<details>
<summary><b>View Solution</b></summary>

Use the following template to create a deployment called `ocean-apd`:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ocean-apd
  name: ocean-apd
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ocean-apd
  strategy: 
   type: RollingUpdate
   rollingUpdate:
     maxUnavailable: 45%
     maxSurge: 55%
  template:
    metadata:
      labels:
        app: ocean-apd
    spec:
      containers:
      - image: kodekloud/webapp-color:v1
        name: webapp-color
```

Now, create the deployment by using the `kubectl create -f` command in the `default` namespace:

```bash
kubectl create -f <FILE-NAME>.yaml
```

After sometime, upgrade the deployment image to `kodekloud/webapp-color:v2`:

```bash
kubectl set image deploy ocean-apd webapp-color=kodekloud/webapp-color:v2
```

And check out the rollout history of the deployment `ocean-apd`:

```bash
kubectl rollout history deploy ocean-apd
```

On the `student-node`, store the revision count to the given file:

```bash
echo "2" > /opt/ocean-revision-count.txt
```

In final task, rollback the deployment image to an old version:

```bash
kubectl rollout undo deployment ocean-apd
```

Verify the image name by using the following command:

```bash
kubectl describe deploy ocean-apd | grep -i image
```

It should be `kodekloud/webapp-color:v1` image.

</details>

**Details (Verification):**

* Is deployment running?
* Is replica set to 2?
* Is maxSurge set to 55%?
* Is maxUnavailable set to 45%?
* Is revision count stored in the file?
* Is rolling back successful?

-----

### Question 7

**Context:** `kubectl config use-context cluster1`

**Task:**
In the initial phase, our team lead requires the deployment of Headlamp utilizing the Helm tool. Headlamp serves as a robust and extensible web-based management interface for Kubernetes clusters, enabling users to manage and monitor cluster resources, deploy applications, troubleshoot issues, and more.

This deployment will be facilitated through the `headlamp` Helm chart on the `cluster1-controlplane` node to fulfill the requirements of our new client.

The chart URL and other specifications are as follows:

1. The chart URL link: `https://kubernetes-sigs.github.io/headlamp/`
2. The chart repository name should be `headlamp`.
3. The release name should be `kubernetes-dashboard`.
4. All the resources should be deployed on the `cd-tool-apd` namespace.

**NOTE:** You have to perform this task from the `student-node`.

<details>
<summary><b>View Solution</b></summary>

In this task, we will use the `helm` commands. Here are the steps:

Add the repository to Helm with the following command:

```bash
helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
```

The `helm repo add` command is used to add a new chart repository to Helm and this allows us to browse and install charts from the new repository using the Helm package manager.

Use the `helm repo ls` command which is used to list all the currently configured Helm chart repositories on Kubernetes cluster.

```bash
helm repo ls 
```

Install the `headlamp` chart on the Kubernetes cluster with the following command:

```bash
helm upgrade --install kubernetes-dashboard headlamp/headlamp -n cd-tool-apd --create-namespace
```

Or

```bash
helm repo update
helm install kubernetes-dashboard headlamp/headlamp -n cd-tool-apd --create-namespace
```

</details>

**Details (Verification):**

* Is Helm repository created?
* Is Helm chart installed?
* Are resources deployed on ns?

-----

### Question 8

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a deployment called `space-20` using the `redis` image to the `aerospace` namespace.

<details>
<summary><b>View Solution</b></summary>

Run the following command:

```bash
kubectl create deployment space-20 --image=redis -n aerospace
```

To cross-verify the deployed resources, run the `kubectl get` command as follows:

```bash
kubectl get pods,deployments -n aerospace
```

</details>

**Details (Verification):**

* Is the deployment created?

-----

## Services and Networking

### Question 9

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a pod with name `pod21-ckad-svcn` using the `nginx:alpine` image in the default namespace and also expose the pod using service `pod21-ckad-svcn` on `port 80`.

Note: Use the imperative command for above scenario.

<details>
<summary><b>View Solution</b></summary>

As the scenario mention to create a pod along with its service with name `pod21-ckad-svcn`, we can use the following imperative command.

```bash
kubectl run pod21-ckad-svcn --image=nginx:alpine --expose --port=80
# Or
kubectl run pod21-ckad-svcn --image=nginx:alpine
kubectl expose pod pod21-ckad-svcn --port=80 --name=pod21-ckad-svcn
```

It will create a pod and also exposes the pod at port 80.

</details>

**Details (Verification):**

* Is the pod `pod21-ckad-svcn` created?
* Is service `pod21-ckad-svcn` created?

-----

### Question 10

**Context:** `kubectl config use-context cluster3`

**Task:**
We have already deployed an application that consists of frontend, backend, and database pods in the `app-ckad` namespace. **Inspect them.**

Your task is to create:

* A service `frontend-ckad-svcn` to expose the frontend pods outside the cluster on `port 31100`.
* A service `backend-ckad-svcn` to make backend pods to be accessible within the cluster.
* A policy `database-ckad-netpol` to limit access to database pods only to backend pods.

<details>
<summary><b>View Solution</b></summary>

* A service `frontend-ckad-svcn` to expose the frontend pods outside the cluster on `port 31100`.

<!-- end list -->

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-ckad-svcn
  namespace: app-ckad
spec:
  selector:
    app: frontend
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 31100
```

* A service `backend-ckad-svcn` to make backend pods to be accessible within the cluster.

<!-- end list -->

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-ckad-svcn
  namespace: app-ckad
spec:
  selector:
    app: backend
  ports:
  - name: http
    port: 80
    targetPort: 80
```

* A policy `database-ckad-netpol` to limit access of database pods only to backend pods.

<!-- end list -->

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-ckad-netpol
  namespace: app-ckad
spec:
  podSelector:
    matchLabels:
      app: database
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
```

</details>

**Details (Verification):**

* Is service `frontend-ckad-svcn` created?
* Is service `backend-ckad-svcn` created?
* Is policy `database-ckad-netpol` created?

### Question 11

**Context:** `kubectl config use-context cluster3`

**Task:**
For this scenario, create a deployment named `hr-web-app-ckad05-svcn` using the image `kodekloud/webapp-color` with `2` replicas.

Expose the `hr-web-app-ckad05-svcn` with service named `hr-web-app-service-ckad05-svcn` on port `30082` on the nodes of the cluster.

**Note:** The web application listens on port `8080`.

<details>
<summary><b>View Solution</b></summary>

Switch to `cluster3` :

```bash
kubectl config use-context cluster3
```

On student-node, use the command:

```bash
kubectl create deployment hr-web-app-ckad05-svcn --image=kodekloud/webapp-color --replicas=2
```

Now we can run the following command to generate a service definition file:

```bash
kubectl expose deployment hr-web-app-ckad05-svcn --type=NodePort --port=8080 --name=hr-web-app-service-ckad05-svcn --dry-run=client -o yaml > hr-web-app-service-ckad05-svcn.yaml
```

Now, in the generated service definition file add the `nodePort` field with the given port number under the `ports` section and create a service.

</details>

**Details (Verification):**

* Is deployment name `hr-web-app-ckad05-svcn`?
* Is the Image `kodekloud/webapp-color` used?
* Does deployment has `2` replicas?
* Is service `hr-web-app-service-ckad05-svcn` created?
* Is service is of type "NodePort"?
* Does service has 2 Endpoints?
* Is service configured for port `8080`?
* Does service use NodePort `30082`?

-----

### Question 12

**Context:** `kubectl config use-context cluster1`

**Task:**
Please use the namespace `nginx-deployment` for the following scenario.

Create a deployment with name `nginx-ckad11` using `nginx` image with `2` replicas. Also expose the deployment via ClusterIP service .i.e. `nginx-ckad11-service` on port 80. Use the label `app=nginx-ckad` for both resources.

Now, create a NetworkPolicy .i.e. `ckad-allow` so that only pods with label `criteria: allow` can access the deployment on port 80 and apply it.

<details>
<summary><b>View Solution</b></summary>

Use the following to create the deployment and the service:

```bash
kubectl apply -f - <<eof
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ckad11
  namespace: nginx-deployment
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
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-ckad11-service
  namespace: nginx-deployment
spec:
  selector:
    app: nginx-ckad
  ports:
    - name: http
      port: 80
      targetPort: 80
  type: ClusterIP
eof
```

To create a NetworkPolicy that only allows pods with labels `criteria: allow` to access the deployment, you can use the following YAML definition:

```bash
kubectl apply -f - <<eof
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ckad-allow
  namespace: nginx-deployment
spec:
  podSelector:
    matchLabels:
      app: nginx-ckad
  ingress:
    - from:
        - podSelector:
            matchLabels:
              criteria: allow
      ports:
        - protocol: TCP
          port: 80
eof
```

</details>

**Details (Verification):**

* Is deployment with `nginx` image created?
* Does deployment created have 2 replicas?
* Is service of type "ClusterIP"?
* Is service configured to use targetPort 80?
* Is network policy - `ckad-allow` created?
* Does policy allows traffic from pod with label `criteria=allow`?
* Pods without mentioned labels cannot access.

-----

## Application Environment, Configuration and Security

### Question 13

**Context:** `kubectl config use-context cluster1`

**Task:**
Update pod `ckad06-cap-aecs` in the namespace `ckad05-securityctx-aecs` to run as root user and with the `SYS_TIME` and `NET_ADMIN` capabilities.

**Note:** Make only the necessary changes. Do not modify the name of the pod.

<details>
<summary><b>View Solution</b></summary>

```bash
kubectl config use-context cluster1
```

Get the pod definition:

```bash
kubectl get -n ckad05-securityctx-aecs pod ckad06-cap-aecs -o yaml > pod-capabilities.yaml
```

Edit the pod definition file to add the capabilities:

```bash
vim pod-capabilities.yaml
```

Update it to match:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ckad06-cap-aecs
  namespace: ckad05-securityctx-aecs
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      # runAsUser: 0 # This is optional as by default container runs as root user if not specified otherwise.
      capabilities:
        add: ["SYS_TIME", "NET_ADMIN"]
```

Replace the pod with the new configuration:

```bash
kubectl replace -f pod-capabilities.yaml --force
```

</details>

**Details (Verification):**

* Are required capabilities added ?

-----

### Question 14

**Context:** `kubectl config use-context cluster3`

**Task:**
Define a Kubernetes custom resource definition (CRD) for a new resource **kind** called `Foo` (plural form - `foos`) in the `samplecontroller.example.com` group.

This CRD should have a version of `v1alpha1` with a schema that includes two properties as given below:

> `deploymentName` (a string type) and `replicas` (an integer type with minimum value of 1 and maximum value of 5).

It should also include a `status` **subresource** which enables retrieving and updating the status of **Foo** object, including the `availableReplicas` property, which is an `integer` type.
The **Foo** resource should be `namespace` **scoped**.

**Note:** We have provided a template `/root/foo-crd-aecs.yaml` for your ease. There are few issues with it so please make sure to incorporate the above requirements before deploying on cluster.

<details>
<summary><b>View Solution</b></summary>

Edit the CRD file:

```bash
vim foo-crd-aecs.yaml
```

Add the required schema definition to the file:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: foos.samplecontroller.example.com
spec:
  group: samplecontroller.example.com
  scope: Namespaced
  names:
    kind: Foo
    plural: foos
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
            # Spec for schema goes here !
            type: object
            properties:
              deploymentName:
                type: string
              replicas:
                type: integer
                minimum: 1
                maximum: 5
          status:
            type: object
            properties:
              availableReplicas:
                type: integer
    # subresources for the custom resource
    subresources:
      # enables the status subresource
      status: {}
```

Apply the CRD:

```bash
kubectl apply -f foo-crd-aecs.yaml
```

</details>

**Details (Verification):**

* Are correct values used for: 'group', 'kind', 'plural' and 'scope' ?
* Is the correct version specified?
* Are correct keys defined of type "string"?
* Are correct keys defined of type "integer"?
* Is the correct range defined for key 'replicas'?
* Is the "status" for foo object correctly configured?
* Are "status" Subresources enabled?

-----

### Question 15

**Context:** `kubectl config use-context cluster1`

**Task:**
We created two **ConfigMaps** ( `ckad02-config1-aecs` and `ckad02-config2-aecs` ) and one **Pod** named `ckad02-test-pod-aecs` in the `ckad02-mult-cm-cfg-aecs` namespace.

Create two environment variables for the above pod with below specifications:

1. `GREETINGS` with the data from configmap `ckad02-config1-aecs`
2. `WHO` with the data from configmap `ckad02-config2-aecs`.

**Note:** Only make the necessary changes. Do not modify other fields of the pod.

<details>
<summary><b>View Solution</b></summary>

```bash
kubectl config use-context cluster1
```

Get the pod definition:

```bash
kubectl get pods -n ckad02-mult-cm-cfg-aecs ckad02-test-pod-aecs -o yaml  > 1-pod-cm.yaml
```

Edit the pod definition file to add the env variables:

```bash
vim 1-pod-cm.yaml
```

Update it to match:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ckad02-test-pod-aecs
  namespace: ckad02-mult-cm-cfg-aecs
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: ["/bin/sh", "-c", "while true; do env | egrep \"GREETINGS|WHO\"; sleep 10; done"]
      env:
        - name: GREETINGS
          valueFrom:
            configMapKeyRef:
              name: ckad02-config1-aecs
              key: greetings.how
        - name: WHO
          valueFrom:
            configMapKeyRef:
              name: ckad02-config2-aecs
              key: man
```

Replace the pod with the new configuration:

```bash
kubectl replace -f 1-pod-cm.yaml --force
```

</details>

**Details (Verification):**

* Is env var for "GREETINGS" configured ?
* Is env var for "WHO" configured ?

-----

### Question 16

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a Kubernetes Pod named `ckad15-memory`, with a container named `ckad15-memory` running the `polinux/stress` image, and configure it to use the following specifications:

* Command: `stress`
* Arguments: `["--vm", "1", "--vm-bytes", "10M", "--vm-hang", "1"]`
* Requested memory: `10Mi`
* Memory limit: `10Mi`

<details>
<summary><b>View Solution</b></summary>

Apply the following YAML to create the pod:

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ckad15-memory
spec:
  containers:
  - name: ckad15-memory
    image: polinux/stress
    resources:
      requests:
        memory: "10Mi"
      limits:
        memory: "10Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "10M", "--vm-hang", "1"]
EOF
```

</details>

**Details (Verification):**

* Is correct container name and image used?
* Is correct command used ?
* Is correct argument used ?
* Is "request" and "limit" for memory configured ?

-----

### Question 17

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a role `configmap-updater` in the `ckad17-auth1` namespace granting the `update` and `get` permissions on `configmaps` resources but restricted to only the `ckad17-config-map` instance of the resource.

<details>
<summary><b>View Solution</b></summary>

You can create the role using the imperative command:

```bash
kubectl create role configmap-updater --namespace=ckad17-auth1 --resource=configmaps --resource-name=ckad17-config-map --verb=update,get 
```

</details>

**Details (Verification):**

* Are correct verbs defined?
* Are correct resource and resource instance defined ?

-----

## Application Observability and Maintenance

### Question 18

**Context:** `kubectl config use-context cluster1`

**Task:**
A pod named `ckad-simplepod-aom` is deployed, but it is not running state. Inspect the reason and make required changes.

<details>
<summary><b>View Solution</b></summary>

Let's debug the issue first:

```bash
kubectl describe pod ckad-simplepod-aom
```

This will show an image pull issue as there is an image typo and we are pulling an incorrect image.
Use the below command for the fixes:

```bash
kubectl edit pod ckad-simplepod-aom
```

And make changes according to YAML solution provided:

```yaml
      image: busybox
      name: pods-simple-container
```

</details>

**Details (Verification):**

* Status: Running
* Is the image issue fixed?

-----

### Question 19

**Context:** `kubectl config use-context cluster1`

**Task:**
Identify the kube api-resources that use `api_version=storage.k8s.io/v1` using kubectl command line interface and store them in `/root/api-version.txt` on student-node.

<details>
<summary><b>View Solution</b></summary>

Use the following command to get details:

```bash
kubectl api-resources --sort-by=kind | grep -i storage.k8s.io/v1  > /root/api-version.txt
```

</details>

**Details (Verification):**

* apiVersion: storage.k8s.io/v1
* Resources identified

-----

### Question 20

**Context:** `kubectl config use-context cluster3`

**Task:**
A manifest file located at `root/ckad-aom.yaml` on **cluster3-controlplane**. Which can be used to create a multi-containers pod. There are issues with the manifest file, preventing resource creation. Identify the errors, fix them and create resource.

**Note:** You can access controlplane by `ssh cluster3-controlplane` if required.

<details>
<summary><b>View Solution</b></summary>

Set context to `cluster3` and SSH to cluster3-controlplane.
When you use the `kubectl create` command, you will see following error:

```bash
kubectl create -f ckad-aom.yaml
error: resource mapping not found for name: "ckad-aom" namespace: "" from "ckad-aom.yaml": no matches for kind "Pod" in version "V1"
ensure CRDs are installed first
```

The error shows us there is something wrong with apiVersion. So change it to `v1`, try again, and check status.

```bash
kubectl get pods
NAME            READY   STATUS             RESTARTS      AGE
ckad-aom   1/2     CrashLoopBackOff   3 (39s ago)   93s
```

Now check for reason using:

```bash
kubectl describe pod ckad-aom
```

We will see that there is a problem with the **nginx container**.
Open yaml file and check in **spec -> nginx container** you can see an error with `mountPath` -> **mountPath: "/var/log"** change it to **mountPath: `/var/log/nginx`** and apply changes.

</details>

**Details (Verification):**

* Pod status
* Used correct apiVersion
* Configured correct mount path

-----

### Question 21

**Context:** `kubectl config use-context cluster2`

**Task:**
Use the manifest file `/root/resourcedefinition.yaml` and extend kubernetes API for **mongodb** service.

But there is an error with the file content. Find the error and fix the issue.

<details>
<summary><b>View Solution</b></summary>

The following error will be shown for creation of CRD:

```bash
error: unable to recognize "resourcedefinition.yaml": no matches for kind "CustomResourceDefinition" in version "k8s.io/v1"
```

Check for apiVersion:

```bash
kubectl api-resources | grep -i crd
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1
```

Replace the correct apiVersion `apiextensions.k8s.io/v1` and extend API with:

```bash
kubectl create -f resourcedefinition.yaml
```

</details>

**Details (Verification):**

* CRD created
