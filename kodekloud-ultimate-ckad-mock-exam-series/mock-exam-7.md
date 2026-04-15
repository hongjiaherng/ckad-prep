# Lab: CKAD Mock Exam 7


## Application Design And Build

### Question 1

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-multi-containers` namespace, we created a multi-container pod named `readiness-multi-containers`.   
  
For some reasons, it failed to run properly (`Init:CrashLoopBackOff`), then review the issue in the pod and try to run it again.  
  
  
**NOTE**: delete and recreate `readiness-multi-containers` pod if needed!

<details>
<summary><b>View Solution</b></summary>

First query the pod and see if it running or not:
```bash
student-node ~ ➜  kubectl get po -n ckad-multi-containers
NAME                         READY   STATUS                  RESTARTS     AGE
readiness-multi-containers   0/1     Init:CrashLoopBackOff   1 (6s ago)   9s
```

You can see its status is `Init:CrashLoopBackOff`, which means something went wrong with the pod.

Check the main container logs:
```bash
kubectl logs readiness-multi-containers -n ckad-multi-containers 
Error from server (BadRequest): container "main-application" in pod "readiness-multi-containers" is waiting to start: PodInitializing
```

The main container can't be started as it waiting for the init container.  
  
  
Check the init container logs:
```bash
student-node ~ ✖ kubectl logs readiness-multi-containers -n ckad-multi-containers init-application
sh: slept: not found
```

The issue here is `slept` command is not found (typo issue), you just need to update the command then the pod will start correctly.  
  
  
The command of init-container should look like below:
```bash
initContainers:
  - command:
    - sh
    - -c
    - sleep 10
```

</details>

**Details (Verification):**

* Is the `readiness-multi-containers` pod running?

-----

### Question 2

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-multi-containers` namespace, create a pod named `healthy-server`, which consists of 2 containers. One main container and one init-container both are running `busybox:1.28` image.   
  
  
Init container should print this message `Initialize application environment!` and then sleep for `10` seconds.  
  
Main container should print this message `The app is running!` and then sleep for `3600` seconds.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create required pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: healthy-server
  namespace: ckad-multi-containers
  labels:
    app.kubernetes.io/name: healthy-server
spec:
  containers:
    - name: server-container
      image: busybox:1.28
      command:
        - sh
        - -c
        - echo The app is running! && sleep 3600
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command:
        - sh
        - -c
        - echo Initialize application environment! && sleep 10
```

</details>

**Details (Verification):**

* Is the `healthy-server` running?
* Does main container run `busybox:1.28` image?
* Does main container run the desired command?
* Does init container run `busybox:1.28` image?
* Does init container run the desired command?

-----

### Question 3

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-pod-design` namespace, start a `ckad-nginx-wiipwlznjy` pod running the `nginx:1.17` image; the container should be named `nginx-custom-annotation`.  
  
  
Configure a custom `annotation` to that pod as below:
```bash
HOMEPAGE: https://kodekloud.com/
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
    run: ckad-nginx-wiipwlznjy
  name: ckad-nginx-wiipwlznjy
  namespace: ckad-pod-design
  annotations: 
    HOMEPAGE: https://kodekloud.com/
spec:
  containers:
  - image: nginx:1.17
    name: nginx-custom-annotation
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then use `kubectl apply -f file_name.yaml` to create the required object.  
  
Alternatively, you can use this command for similar outcome:
```bash
kubectl run ckad-nginx-wiipwlznjy -n ckad-pod-design --image=nginx:1.17 --annotations HOMEPAGE=https://kodekloud.com
```

</details>

**Details (Verification):**

* Is pod `ckad-nginx-wiipwlznjy` running?
* Is the pod's container named `nginx-custom-annotation`?
* Is the pod's annotation configured correctly?

-----

### Question 4

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad-job` namespace, create a job named `pi` that simply computes a `π` (pi) to 2000 places and prints it out.

This job should be configured to retry maximum `3 times` before marking this job failed, and the duration of this job should not exceed `100 seconds`.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create job:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
  namespace: ckad-job
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

You can verify the output by running below command against the job's pod (noted that the pod name will be different):  
  
Identify pod name by this command:  
 `kubectl get pod -n ckad-job -l job-name=pi`
```bash
kubectl logs pi-4zvcc -n ckad-job 
3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141273724587006606315588174881520920962829254091715364367892590360011330530548820466521384146951941511609433057270365759591953092186117381932611793105118548074462379962749567351885752724891227938183011949129833673362440656643086021394946395224737190702179860943702770539217176293176752384674818467669405132000568127145263560827785771342757789609173637178721468440901224953430146549585371050792279689258923542019956112129021960864034418159813629774771309960518707211349999998372978049951059731732816096318595024459455346908302642522308253344685035261931188171010003137838752886587533208381420617177669147303598253490428755468731159562863882353787593751957781857780532171226806613001927876611195909216420198938095257201065485863279
```

</details>

**Details (Verification):**

* Is the job `pi` created?
* Is the container image `perl:5.34.0`?
* Does the job run the required command?
* Is the maximum retry well configured?
* Is the job maximum duration well configured?

-----


## Application Deployment

### Question 5

**Context:** `kubectl config use-context cluster2`

**Task:**
The team `Garuda` has deployed one application in the `testing-apd` namespace. The testing was done, and the team wants you to delete that release. Find out the release name and delete it.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster2
```

On the cluster2, testing application is running on the `testing-apd` namespace, which we must delete because it is consuming resources.
```bash
helm ls -n testing-apd


helm uninstall -n testing-apd image-scanner
```

</details>

**Details (Verification):**

* Are testing resources deleted?

-----

### Question 6

**Context:** `kubectl config use-context cluster3`

**Task:**
We have deployed two applications called `circle-apd` and `square-apd` on the `default` namespace using the `kodekloud/webapp-color:v1` and `kodekloud/webapp-color:v2`.   
  
We have done all the tests and do not want `circle-apd` deployment to receive traffic from the `foundary-svc` service anymore. So, route all the traffic to another existing deployment.

Do change the service specifications to route traffic to the `square-apd` deployment.

You can test the application from the terminal by running the `curl` command with the following syntax: -
```bash
curl http://cluster3-controlplane:NODE-PORT
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
kubectl config use-context cluster3
```

In this task, we will use the `kubectl` command. Here are the steps: -

1. Use the `kubectl get` command to list all the given resources: -
```bash
kubectl get deploy,svc -n default
```

2. Identify the created deployment and service names.
3. Now, check the label used in the service `selector` field for the deployment named `circle-apd`.
```bash
kubectl describe service foundary-svc
```

4. Check the labels for the deployment named `square-apd` deployment: -
```bash
kubectl get deploy square-apd -oyaml
```

5. Update the selector label of the service.
```bash
kubectl edit service foundary-svc
```

By default, it will open a `VI` editor. After updating the selector labels. Press `ESC` and type `:wq`. It will save the changes and quit the editor.

</details>

**Details (Verification):**

* Is the application passed the test?
* Is the service updated?

-----

### Question 7

**Context:** `kubectl config use-context cluster3`

**Task:**
The deployment called `foundary-apd` inside the `blue-apd` namespace on `cluster3` has undergone several, routine, rolling updates and rollbacks.   
   
  
Inspect the `revision 3` of this deployment and store the image name that was used in this revision in the `/root/records/foundry-revision-book.txt` file on the `student-node`.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster3
```

In this task, we will use the `kubectl get` and `kubectl rollout` commands. Here are the steps: -

1. To check the deployment under the `blue-apd` namespace, run the following command: -
```bash
kubectl get deploy -n blue-apd
```

In the task, we have been given to check the revisions. For this, we use the `kubectl rollout history` command to view the revision history of the deployment called `foundary-apd`. Run the following command: -
```bash
kubectl rollout history deployment -n blue-apd foundary-apd
```

We will see all the revisions of the given deployment.

Now, inspect the `revision 3` by using the `--revision` option: -
```bash
kubectl rollout history deployment -n blue-apd foundary-apd --revision=3
```

Under the `Containers` section, we will see the `image` name.

On the `student-node`, save that image name in the given file `/root/records/foundry-revision-book.txt`: -
```bash
echo "ubuntu:1.21" > /root/records/foundry-revision-book.txt
```

If the `records` directory is absent, use the `mkdir` command to create this directory.

</details>

**Details (Verification):**

* Is deployment running?
* Is revision 3 image saved to a file?

-----

### Question 8

**Context:** `kubectl config use-context cluster1`

**Task:**
One application, `webpage-server-01`, is deployed on the Kubernetes cluster by the Helm tool. Now, the team wants to deploy a new version of the application by replacing the existing one. A new version of the helm chart is given in the `/root/new-version` directory on the `student-node`. Validate the chart before installing it on the Kubernetes cluster.   
  
  
Use the `helm` command to validate and install the chart. After successfully installing the newer version, uninstall the older version.

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

</details>

**Details (Verification):**

* Is the new version app deployed?
* Is the old version app uninstalled?

-----

### Question 9

**Context:** `kubectl config use-context cluster3`

**Task:**
An application called `shipping-api` is running on cluster3. In the weekly meeting, the team decides to upgrade the version of the existing image to `nginx:perl` and wants to store the new version of the image in a file `/root/records/new-image-records.txt` on the `cluster3-controlplane` instance.

After upgrading the image version, to increase the availability and performance of the application, scale the deployment to `2`, and ensure that the targeted pod is running.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster3
```

In this task, we will use the `kubectl describe`, `kubectl get`, `kubectl set` and `kubectl scale` commands. Here are the steps: -

1. To check all the deployments in all the namespaces in the `cluster3`, we would have to run the following command:
```bash
kubectl get deployments -A
```

Inspect all the deployments.

2. We can see that one of the deployment's names is `shipping-api` and deployed on `ckad9-uat` namespace. Use the `kubectl describe` command to get detailed information of that deployment: -
```bash
kubectl describe -n ckad9-uat deploy shipping-api
```

The output of the `kubectl describe` command will provide you with a detailed description of the deployment, including its name, namespace, creation time, labels, replicas, and the Docker image being used.

3. In the previous command, we can see the container and image name under the `Pod Template` spec. Use the `kubectl set` command to update the image of that container as follows:
```bash
kubectl set image -n ckad9-uat deploy shipping-api shipping-api=nginx:perl
```

After running the above command, Kubernetes will automatically update the `shipping-api` container with the new image, and create a new replica of the resource with the updated image.   
The old replica will be deleted once the new one is up and running.

4. Now, `SSH` to the `cluster3-controlplane` node and use `echo` command to add this new image to a file at `/root/records/new-image-records.txt`: -
```bash
echo "nginx:perl" > /root/records/new-image-records.txt
```

If the `records` directory is absent, use the `mkdir` command to create this directory.

> NOTE: - To exit from any node, type `exit` on the terminal or press `CTRL + D`.

5. Now, run the `kubectl scale` command to scale the deployment to `2`:
```bash
kubectl scale deployment -n ckad9-uat shipping-api --replicas=2
```

Cross-verify the scaled deployment by using the `kubectl get` command:
```bash
kubectl get deployments,pods -n ckad9-uat
```

</details>

**Details (Verification):**

* Is the image updated?
* Is the deployment scaled?
* Did the image get stored in a file?

-----

### Question 10

**Context:** `kubectl config use-context cluster1`

**Task:**
One co-worker deployed an nginx helm chart on the `cluster1` server called `bitnami`. A new update is pushed to the helm chart, and the team wants you to update the helm repository to fetch the new changes.   
   
  
After updating the helm chart, upgrade the helm chart version to `18.3.0` and increase the replica count to `2`.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster1
```

In this task, we will use the `kubectl` and `helm` commands. Here are the steps: -

1. Log in to the `cluster1-controlplane` node first and use the `helm ls` command to list all the releases installed using Helm in the Kubernetes cluster.
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
helm repo update bitnami -n ckad10-finance-ns
```

The above command updates the local cache of available charts from the configured chart repositories.

5. The `helm search` command searches for all the available charts in a specific Helm chart repository. In our case, it's the nginx helm chart.
```bash
helm search repo bitnami/nginx -n ckad10-finance-ns -l | head -n30
```

The `-l` or `--versions` option is used to display information about all available chart versions.

6. Upgrade the helm chart to `18.3.0` and also, increase the replica count of the deployment to `2` from the command line. Use the `helm upgrade` command as follows: -
```bash
helm upgrade bitnami bitnami/nginx -n ckad10-finance-ns --version=18.3.0 --set replicaCount=2
```

7. After upgrading the chart version, you can verify it with the following command: -
```bash
helm ls -n ckad10-finance-ns
```

Look under the `CHART` column for the chart version.

8. Use the `kubectl get` command to check the replicas of the deployment: -
```bash
kubectl get deploy -n ckad10-finance-ns
```

The available count `2` is under the `AVAILABLE` column.

</details>

**Details (Verification):**

* Is deployment running?
* Is the chart version upgraded?
* Is the deployment scaled?

-----


## Services And Networking

### Question 11

**Context:** `kubectl config use-context cluster3`

**Task:**
We have already deployed an ingress resource in the `app-space` namespace.   
  
  
But for better SEO practices, you are requested to change the URLs at which the applications are made available.  
  
  
Change the path of the video application to make it available at `/stream`.

<details>
<summary><b>View Solution</b></summary>

- To change the path of the service, you need to edit the ingress
  
Edit the ingress using `kubectl edit -n app-space ingress ingress-resource-svcn`.  
- Edit the /watch to make it as /stream.
```yaml
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /watch  #change path to /stream
        pathType: Prefix
status:
  loadBalancer:
    ingress:
    - ip: 10.108.22.212
```

</details>

**Details (Verification):**

* Does path /wear select `wear-service`?
* Does path /stream select video application?
* Is backend port for /wear service correct?
* Is backend port for /stream service service?

-----

### Question 12

**Context:** `kubectl config use-context cluster3`

**Task:**
We have deployed some pods in the namespaces `ckad-alpha` and `ckad-beta`.

You need to create a NetworkPolicy named `ns-netpol-ckad` that will restrict all Pods in Namespace `ckad-alpha` to only have outgoing traffic to Pods in Namespace `ckad-beta` . Ingress traffic should not be affected.

However, the NetworkPolicy you create should allow egress traffic on port 53 TCP and UDP.

<details>
<summary><b>View Solution</b></summary>

The following manifest will restrict all the pods in `ckad-alpha` namespace to only have egress traffic to pods in namespace `ckad-beta`. But it will allow egress on port 53.
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ns-netpol-ckad
  namespace: ckad-alpha
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
  - to:
     - namespaceSelector:
        matchLabels:
         kubernetes.io/metadata.name: ckad-beta
```

</details>

**Details (Verification):**

* Is network policy `ns-netpol-ckad` created?
* Is gress traffic to `ckad-beta` namespace allowed?
* Is egress to other namespaces restricted?
* Is egress from port 53 allowed?
* Is ingress traffic allowed?

-----

### Question 13

**Context:** `kubectl config use-context cluster2`

**Task:**
We have deployed an application in the `green-space` namespace. we also deployed the ingress controller and the ingress resource.   
  
  
However, currently, the ingress controller is not working as expected. Inspect the ingress definitions and troubleshoot the issue so that the services are accessible as per the ingress resource definition.

<details>
<summary><b>View Solution</b></summary>

Check the status of the ingress, pods, and application related services.
```bash
cluster2-controlplane ~ ➜  k get pods -n ingress-nginx 
NAME                                        READY   STATUS      RESTARTS      AGE
ingress-nginx-admission-create-l6fgw        0/1     Completed   0             11m
ingress-nginx-admission-patch-sfgc4         0/1     Completed   0             11m
ingress-nginx-controller-5f8964959d-278rc   0/1     Error       2 (26s ago)   29s
```

You would see an Error or CrashLoopBackOff in the ingress-nginx-controller. Inspect the logs of the controller pod.
```bash
cluster2-controlplane ~ ✖ k logs -n ingress-nginx ingress-nginx-controller-5f8964959d-278rc 
-------------------------------------------------------------------------------
--------
F0316 08:03:28.111614      57 main.go:83] No service with name default-backend-service found in namespace default:

-------
```

You see an error msg saying "No service with name default-backend-service found in namespace default".

* We don't have the service with that name in the default namespace, so we need to edit the ingress controller deployment to use the service that we have .i.e. `default-backend-service` in the `green-space` namespace.
* To create the controller deployment with correct backend service, first save the deployment in a file, delete the controller deployment, edit the file and create the deployment.
* Save the deployment in file
```bash
k get -n ingress-nginx deployments.apps ingress-nginx-controller -o yaml >> ing-control.yaml
```

* Delete the deployment.   
  `k delete -n ingress-nginx deploy ingress-nginx-controller`
* Edit the file to match the correct service.
```yaml
spec:
   containers:
   - args:
     - /nginx-ingress-controller
     - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
     - --election-id=ingress-controller-leader
     - --watch-ingress-without-class=true
     - --default-backend-service=green-space/default-backend-service   #Changed to correct namespace
     - --controller-class=k8s.io/ingress-nginx
     - --ingress-class=nginx
     - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
     - --validating-webhook=:8443
     - --validating-webhook-certificate=/usr/local/certificates/cert
     - --validating-webhook-key=/usr/local/certificates/key
```

* Apply the manifest, it should be up and running.
* You can verify the application is running via going to host **cluster2-controlplane**: `ssh cluster2-controlplane` and curl request to the application endpoint
```bash
# App Video Service
curl -H "Host: app-video.localhost" http://localhost:30080

# App Wear Service 
curl -H "Host: app-wear.localhost" http://localhost:30080
```

</details>

**Details (Verification):**

* Is the ingress controller issue fixed?
* Is the app-wear application accessible using Ingress with Host app-wear.localhost?
* Is the app-video application accessible using Ingress with Host app-video.localhost?

-----


## Application Environment,  Configuration And Security

### Question 14

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad18-rq-ns-aecs` namespace, create a **ResourceQuota** called `ckad18-rq-aecs`. This ResourceQuota should have the following specifications for **CPU** and **memory**:

> Requests: CPU: `50m`, Memory: `100Mi`

> Limits: CPU: `100m`, Memory: `200Mi`

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl create ns ckad18-rq-ns-aecs
namespace/ckad18-rq-ns-aecs created

student-node ~ ➜  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ckad18-rq-aecs
  namespace: ckad18-rq-ns-aecs
spec:
  hard:
    requests.cpu: "50m"
    requests.memory: 100Mi
    limits.cpu: "100m"
    limits.memory: 200Mi
EOF

resourcequota/ckad18-rq-aecs created

student-node ~ ➜  kubectl get resourcequotas -n ckad18-rq-ns-aecs 
NAME           AGE   REQUEST                                         LIMIT
mem-cpu-aecs   1m   requests.cpu: 0/50m, requests.memory: 0/100Mi   limits.cpu: 0/100m, limits.memory: 0/200Mi


# Validating if the resource quotas working or not !!
student-node ~ ➜  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
  namespace: ckad18-rq-ns-aecs
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "210Mi"
        cpu: "200m"
      requests:
        memory: "100Mi"
        cpu: "50m"
EOF

Error from server (Forbidden): error when creating "STDIN": pods "quota-mem-cpu-demo" is forbidden: exceeded quota: mem-cpu-aecs, requested: limits.cpu=200m,limits.memory=210Mi, used: limits.cpu=0,limits.memory=0, limited: limits.cpu=100m,limits.memory=200Mi
```

</details>

**Details (Verification):**

* Is ResourceQuota created with correct specifications?

-----

### Question 15

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a Kubernetes **LimitRange** object named `ckad15-memlt-aecs`. This object limits the amount of memory each container in the namespace `ckad15-memlt-ns-aecs` can use.   
  
  
The default memory limit should be `512Mi`, and the default memory request should be `256Mi`. Ensure that the manifest applies to container resources only.  
  
  
Note: You may need to create or delete resources to complete this task.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl create namespace ckad15-memlt-ns-aecs
namespace/ckad15-memlt-ns-aecs created

student-node ~ ➜  cat << EOF | kubectl apply -f - -n ckad15-memlt-ns-aecs
apiVersion: v1
kind: LimitRange
metadata:
  name: ckad15-memlt-aecs
  namespace: ckad15-memlt-ns-aecs
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
EOF

limitrange/ckad15-memlt-aecs created
```

</details>

**Details (Verification):**

* Is "LimitRange" object created with correct specifications?

-----

### Question 16

**Context:** `kubectl config use-context cluster2`

**Task:**
Please fix any possible mistakes in the manifest file located at `/root/ckad11-obj-aecs.yaml`, which contains a sample object created from a custom resource definition.  
  
**Note**: Create the object using the above manifest when done.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  kubectl apply -f /root/ckad11-obj-aecs.yaml
error: error parsing /root/ckad11-obj-aecs.yaml: error converting YAML to JSON: yaml: line 7: mapping values are not allowed in this context

student-node ~ ✖ vim /root/ckad11-obj-aecs.yaml

student-node ~ ➜  cat /root/ckad11-obj-aecs.yaml
apiVersion: apps/v1
kind: MyCustomResource
metadata:
  name: ckad11-my-custom-resource-aecs
spec:
  name: John  #1 Fixed the indentation issue
  age: -10
status:
  phase: Invalid

student-node ~ ➜  kubectl apply -f /root/ckad11-obj-aecs.yaml
error: unable to recognize "/root/ckad11-obj-aecs.yaml": no matches for kind "MyCustomResource" in version "apps/v1"

student-node ~ ✖ kubectl api-resources | grep -i MyCustomResource
mycustomresources                 mcr          example.com/v1                         true         MyCustomResource

student-node ~ ➜  vim /root/ckad11-obj-aecs.yaml

student-node ~ ➜  cat /root/ckad11-obj-aecs.yaml
apiVersion: example.com/v1  #2 fixed the apiVersion used
kind: MyCustomResource
metadata:
  name: ckad11-my-custom-resource-aecs
spec:
  name: John  #1 Fixed the indentation issue
  age: -10
status:
  phase: Invalid

student-node ~ ➜  kubectl apply -f /root/ckad11-obj-aecs.yaml
The MyCustomResource "ckad11-my-custom-resource-aecs" is invalid: 
* spec.age: Invalid value: -10: spec.age in body should be greater than or equal to 5
* status.phase: Unsupported value: "Invalid": supported values: "Pending", "Running", "Completed"

student-node ~ ✖ vim /root/ckad11-obj-aecs.yaml

student-node ~ ➜  cat /root/ckad11-obj-aecs.yaml
apiVersion: example.com/v1  #2 fixed the apiVersion used
kind: MyCustomResource
metadata:
  name: ckad11-my-custom-resource-aecs
spec:
  name: John  #1 Fixed the indentation issue
  age: 5 #3 use the valid entries
status:
  phase: Running #4 use the valid entries

student-node ~ ➜  k apply -f /root/ckad11-obj-aecs.yaml
mycustomresource.example.com/ckad11-my-custom-resource-aecs created
```

</details>

**Details (Verification):**

* Is resource object created ?

-----

### Question 17

**Context:** `kubectl config use-context cluster1`

**Task:**
Update `secret/ckad07-sec-cr-aecs` in the `default` namespace with one additional credential stating the **Message of the day** as given below:  
  
  
Name: `ckad07-sec-cr-aecs`  
  
Creds:  
key: `motd`, value: `We make DevOps shine!`  
  
  
Note: Make only the necessary changes. Do not modify other fields of the secret.

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  k get secret/ckad07-sec-cr-aecs -o yaml > secret-motd.yaml

student-node ~ ➜  echo 'We make DevOps shine!' | base64 
V2UgbWFrZSBEZXZPcHMgc2hpbmUhCg==

student-node ~ ➜  vim secret-motd.yaml

student-node ~ ➜  cat secret-motd.yaml 
apiVersion: v1
data:
  eligibility: RGV2T3BzR3V5cw==
  supporters: S29kZUtsb3VkIFRlYW0=
  motd: V2UgbWFrZSBEZXZPcHMgc2hpbmUhCg==  #added
kind: Secret
metadata:
  name: ckad07-sec-cr-aecs
  namespace: default
type: Opaque

student-node ~ ➜  k apply -f secret-motd.yaml 
secret/ckad07-sec-cr-aecs configured

student-node ~ ➜  kubectl get secret/ckad07-sec-cr-aecs -o go-template='{{.data.motd | base64decode}}'
We make DevOps shine!
```

</details>

**Details (Verification):**

* Is new key-value added to secret ?

-----

### Question 18

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a pod named `ckad18-burstable-pod` in namespace `ckad18-qosc-ns` with image `nginx` and container name `ckad18-burstable-pod`.   
  
Define other fields such that the Pod is configured to use the Quality of Service (QoS) class of `Burstable`.  
  
  
Also retrieve the `name` and `QoS` class of each Pod in the namespace `ckad18-qosc-ns` in the below format and save the output to a file named `qos_status_aecs` in the `/root` directory.

**Format**:
```bash
NAME    QOS
pod-1   qos_class
pod-2   qos_class
```

<details>
<summary><b>View Solution</b></summary>

```bash
student-node ~ ➜ kubectl config use-context cluster2
Switched to context "cluster2".

student-node ~ ➜  cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: ckad18-burstable-pod
  namespace: ckad18-qosc-ns
spec:
  containers:
  - name: ckad18-burstable-pod
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
EOF

pod/ckad18-burstable-pod created

student-node ~ ➜  kubectl --namespace=ckad18-qosc-ns get pod --output=custom-columns="NAME:.metadata.name,QOS:.status.qosClass"
NAME                QOS
ckad18-besteffort-pod   BestEffort
ckad18-guaranteed-pod   Guaranteed
ckad18-burstable-pod   Burstable

student-node ~ ➜  kubectl --namespace=ckad18-qosc-ns get pod --output=custom-columns="NAME:.metadata.name,QOS:.status.qosClass" > /root/qos_status_aecs
```

</details>

**Details (Verification):**

* Is the pod created with the "QoS" class of 'Burstable'?
* Is the QoS class of each pod retrieved in the format mentioned?

-----


## Application Observability And Maintenance

### Question 19

**Context:** `kubectl config use-context cluster1`

**Task:**
Three pods `fury,sheldon and flash` were created on `cluster1`. Of the three pods, identify the following and copy them to below file,

* Pod with high memory usage
* **Memory** limit configured to the identified pod.  
    
  copy them as `Podname,Memorylimit` to `/root/pod-metrics` file on student-node.

<details>
<summary><b>View Solution</b></summary>

Use the following command to find the pod metrics
```bash
kubectl top pods
```

Use below command to view the resource limits of pod .
```bash
kubectl get pod fury -o json | jq -r '.spec.containers[].resources.limits.memory'
```

Write the content to`/root/pod-metrics` file.

</details>

**Details (Verification):**

* Metrics logs
* Resourcelimit

-----

### Question 20

**Context:** `kubectl config use-context cluster2`

**Task:**
A `ckad-init-pod-aom` has been deployed in `cluster2` in the `ckad-23` namespace.   
  
There is a problem with the pod; it is not in running state find the cause for its present state and modify the content of the pod if required to bring it to the `running` state.

<details>
<summary><b>View Solution</b></summary>

Set the context to cluster 2
```bash
kubectl config use-context cluster2
```

Check for status of pod in `ckad-23` namespace
```bash
student-node ~ ➜  kubectl get pods -n ckad-23
NAME            READY   STATUS            RESTARTS   AGE
ckad-init-pod   0/1     Init:StartError   0          72s
```

Error is indicating that there is problem with init container.   
 use
```bash
kubectl describe pod ckad-init-pod-aom -n ckad-23
```

and observe for following error
```bash
Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process
```

get the yaml file for the pod and add `/bin/sh` interpreter to the initcontainer command and now use this yaml to create a pod in `ckad-23 namespace` on cluster 2.

</details>

**Details (Verification):**

* Modified init container command
* Pod status is running

-----

### Question 21

**Context:** `kubectl config use-context cluster2`

**Task:**
Pod manifest file is already given under the `/root/` directory called `ckad21-fluentd-pod.yaml`.   
  
There is error with manifest file correct the file and create resource.

<details>
<summary><b>View Solution</b></summary>

You will see following error
```bash
student-node ~ ➜  kubectl create -f ckad21-fluentd-pod.yaml
Error from server (BadRequest): error when creating "ckad21-fluentd-pod.yaml": Pod in version "v1" cannot be handled as a Pod.
```

Use the following yaml file and create resource
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fluentd-elasticsearch
spec:
  containers:
    - command:
        - sleep
        - "3600"
      image: alpine
      name: pods-simple-container
```

</details>

**Details (Verification):**

* Pod is up and running

-----
