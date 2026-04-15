# Lab: CKAD Mock Exam 4


## Application Design And Build

### Question 1

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-pod-design` namespace, create a pod named `privileged-pod` that runs the `nginx:1.17` image, and the container should be run in `privileged` mode.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create the desired pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: privileged-pod
  name: privileged-pod
  namespace: ckad-pod-design
spec:
  containers:
  - image: nginx:1.17
    name: privileged-pod
    resources: {}
    securityContext:
      privileged: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Or in short, use this kubectl command for the same output:
```bash
kubectl run privileged-pod --image=nginx:1.17 --privileged=true -n ckad-pod-design
```

</details>

**Details (Verification):**

* Is pod `privileged-pod` running?
* Is the container image is `nginx:1.17`?
* Is the container in privileged mode?

-----

### Question 2

**Context:** `kubectl config use-context cluster1`

**Task:**
In the `ckad-job` namespace, create a job named `very-long-pi` that simply computes a `π` (pi) to 1024 places and prints it out.

This job should be configured to retry maximum `5 times` before marking this job failed, and the duration of this job should not exceed `100 seconds`.

The job should use the container image `perl:5.34.0`.

The container should run a Perl command that calculates `π` (pi) to 1024 decimal places using:
```bash
perl -Mbignum=bpi -wle 'print bpi(1024)'
```

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create job:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: very-long-pi
  namespace: ckad-job
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(1024)"]
      restartPolicy: Never
```

You can verify the output by running below command against the job's pod (noted that the pod name will be different):  
  
Identify pod name by this command:  
 `kubectl get pod -n ckad-job -l job-name=very-long-pi`
```bash
kubectl logs very-long-pi-4zvcc -n ckad-job 
3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141273724587006606315588174881520920962829254091715364367892590360011330530548820466521384146951941511609433057270365759591953092186117381932611793105118548074462379962749567351885752724891227938183011949129833673362440656643086021394946395224737190702179860943702770539217176293176752384674818467669405132000568127145263560827785771342757789609173637178721468440901224953430146549585371050792279689258923542019956112129021960864034418159813629774771309960518707211349999998372978049951059731732816096318595024459455346908302642522308253344685035261931188171010003137838752886587533208381420617177669147303598253490428755468731159562863882353787593751957781857780532171226806613001927876611195909216420198938095257201065485863279
```

</details>

**Details (Verification):**

* Is the job `very-long-pi` created?
* Is the container image `perl:5.34.0`?
* Does the job run the required command?
* Is the maximum retry well configured?
* Is the job maximum duration well configured?

-----

### Question 3

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a `persistent volume` called `red-pv-ckad03-str` of type: `hostPath` and capacity: `100Mi`.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create required volume:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: red-pv-ckad03-str
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /opt/red-pv-ckad03-str
```

</details>

**Details (Verification):**

* Is the persistent volume `red-pv-ckad03-str` created correctly?

-----

### Question 4

**Context:** `kubectl config use-context cluster2`

**Task:**
In the `ckad-multi-containers` namespaces, create a `ckad-web-pod` pod with `sidecar` container that matches the following requirements.   
  
  
Pod has an `emptyDir` volume named `my-vol`.  
  
  
The main container named `web-app`, runs `nginx:1.16` image. This container mounts the `my-vol` volume at `/usr/share/nginx/html` path.  
  
  
The sidecar container named `log-container`, runs `alpine` image. This container mounts the `my-vol` volume at `/var/log` path.   
  
Every 5 seconds, this container should write the current date along with greeting message `Hi I am from Sidecar container` to `index.html` in the `my-vol` volume.

<details>
<summary><b>View Solution</b></summary>

Use below YAML to create the desired pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: ckad-multi-containers
  name: ckad-web-pod
spec:
  containers:
    - image: nginx:1.16
      name: web-app
      resources: {}
      ports:
        - containerPort: 80
      volumeMounts:
        - name: my-vol
          mountPath: /usr/share/nginx/html
  initContainers: 
     - image: alpine
       restartPolicy: Always
       command:
          - /bin/sh
          - -c
          - while true; do echo $(date -u) Hi I am from Sidecar container >> /var/log/index.html; sleep 5;done
       name: log-container
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
  
  
`kubectl exec -n ckad-multi-containers ckad-web-pod --container web-app -- cat /usr/share/nginx/html/index.html`  
  
  
Similar logs should be displayed as below:
```
Mon Mar 20 06:02:07 UTC 2023 Hi I am from Sidecar container
Mon Mar 20 06:02:12 UTC 2023 Hi I am from Sidecar container
Mon Mar 20 06:02:17 UTC 2023 Hi I am from Sidecar container
Mon Mar 20 06:02:22 UTC 2023 Hi I am from Sidecar container
Mon Mar 20 06:02:27 UTC 2023 Hi I am from Sidecar container
Mon Mar 20 06:02:32 UTC 2023 Hi I am from Sidecar container
Mon Mar 20 06:02:37 UTC 2023 Hi I am from Sidecar container
Mon Mar 20 06:02:42 UTC 2023 Hi I am from Sidecar container
```

</details>

**Details (Verification):**

* Is the main container running?
* Is the pod volume well configured?
* Is the correct message logged to web-app?

-----


## Application Deployment

### Question 5

**Context:** `kubectl config use-context cluster2`

**Task:**
A web application running on cluster2 called `robox-west-apd` on the `fusion-apd-x1df5` namespace. The Ops team has created a new service account with a set of permissions for this web application. Update the newly created SA for this deployment.   
  
  
Also, change the strategy type to `Recreate`, so it will delete all the pods immediately and update the newly created SA to all the pods.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster2
```

In this task, we will use the `kubectl` command. Here are the steps: -

1. Use the `kubectl get` command to list all the given resources: -
```bash
kubectl get po,deploy,sa,ns -n fusion-apd-x1df5
```

Here `-n` option stands for `namespace`, which is used to specify the namespace.

The above command will list all the resources from the `fusion-apd-x1df5` namespace.

2. Inspect the service account is used by the pods/deployment.
```bash
kubectl get deploy -n fusion-apd-x1df5 robox-west-apd -oyaml
```

The deployment is using the default service account.

3. Now, use the `kubectl get` command to retrieves the YAML definition of a deployment named `robox-west-apd` and save it into a file.
```bash
kubectl get deploy -n fusion-apd-x1df5 robox-west-apd -o yaml > <FILE-NAME>.yaml
```

4. Open a `VI` editor. Make the necessary changes and save it. It should look like this: -
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    global-kgh: robox-west-apd
  name: robox-west-apd
  namespace: fusion-apd-x1df5
spec:
  replicas: 3
  selector:
    matchLabels:
      global-kgh: robox-west-apd
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        global-kgh: robox-west-apd
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: robox-container
      serviceAccountName: galaxy-apd-xb12
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

### Question 6

**Context:** `kubectl config use-context cluster1`

**Task:**
On the `cluster1`, the team has installed multiple helm charts on a different namespace. By mistake, those deployed resources include one of the vulnerable images called `kodekloud/click-counter:latest`. Find out the release name and uninstall it.

<details>
<summary><b>View Solution</b></summary>



</details>


-----

### Question 7

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a deployment called `rocket-apd20` using the `nginx:alpine` image to the `aerospace-xgh12fd` namespace.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster2
```

Run the following command: -
```bash
kubectl create deployment rocket-apd20 --image=nginx:alpine -n aerospace-xgh12fd
```

To cross-verify the deployed resources, run the `kubectl get` command as follows: -
```bash
kubectl get pods,deployments -n aerospace-xgh12fd
```

</details>

**Details (Verification):**

* Is the deployment created?

-----

### Question 8

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a new deployment called `web-deployment` in the default namespace using the image `kodekloud/webapp-color:v1`.   
 Use the following specs for the deployment:   
  
  
**1.**  Replica count should be `2`.   
  
**2.**  Set the Max Unavailable to `35%` and Max Surge to `65%`.   
  
**3.**  Create the deployment and ensure all the pods are ready.   
  
**4.**  After successful deployment, upgrade the deployment image to `kodekloud/webapp-color:v2` and inspect the deployment rollout status.   
  
**5.**  Check the rolling history of the deployment and on the `student-node`, save the `current` revision count number to the `/opt/ocean-revision-count.txt` file.   
  
**6.**  Finally, perform a rollback and revert the deployment image to the older version.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster1
```

Use the following template to create a deployment called `web-deployment`: -
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web-deployment
  name: web-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-deployment
  strategy: 
   type: RollingUpdate
   rollingUpdate:
     maxUnavailable: 35%
     maxSurge: 65%
  template:
    metadata:
      labels:
        app: web-deployment
    spec:
      containers:
      - image: kodekloud/webapp-color:v1
        name: webapp-color
```

Now, create the deployment by using the `kubectl create -f` command in the `default` namespace: -
```bash
kubectl create -f <FILE-NAME>.yaml
```

After sometime, upgrade the deployment image to `kodekloud/webapp-color:v2`: -
```bash
kubectl set image deploy web-deployment webapp-color=kodekloud/webapp-color:v2
```

And check out the rollout history of the deployment `web-deployment`: -
```bash
kubectl rollout history deploy web-deployment
deployment.apps/web-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

> NOTE: - Revision count is `2`. In your lab, it could be different.

On the `student-node`, store the revision count to the given file: -
```bash
echo "2" > /opt/ocean-revision-count.txt
```

In final task, rollback the deployment image to an old version: -
```bash
kubectl rollout undo deployment web-deployment
```

Verify the image name by using the following command: -
```bash
kubectl describe deploy web-deployment | grep -i image
```

It should be `kodekloud/webapp-color:v1` image.

</details>

**Details (Verification):**

* Is deployment running?
* Is replica set to 2?
* Is maxSurge set to 65%?
* Is maxUnavailable set to 35%?
* Is revision count stored in the file?
* Is rolling back successful?

-----

### Question 9

**Context:** `kubectl config use-context cluster2`

**Task:**
On cluster2, a new deployment called `cube-alpha-apd` has been created in the `alpha-ns-apd` namespace using the image `kodekloud/webapp-color:v2`. This deployment will test a newer version of the `alpha` app.

Configure the deployment in such a way that the `alpha-apd-service` service routes less than `40%` of traffic to the new deployment.   
  
  
 **NOTE: -**  Do not increase the replicas of the `ruby-alpha-apd` deployment.

<details>
<summary><b>View Solution</b></summary>

Run the following command to change the context: -
```bash
kubectl config use-context cluster2
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


## Services And Networking

### Question 10

**Context:** `kubectl config use-context cluster3`

**Task:**
We have deployed an application in the `green-space` namespace. we also deployed the ingress controller and the ingress resource.   
  
  
However, currently, the ingress controller is not working as expected. Inspect the ingress definitions and troubleshoot the issue so that the services are accessible as per the ingress resource definition.

Also, update the path for the `app-wear-service` to `/app-wear` and `app-video-service` to `/app-video`.

<details>
<summary><b>View Solution</b></summary>

- Check the status of the ingress, pods, and application related services.
```
cluster3-controlplane ~ ➜  k get pods -n ingress-nginx 
NAME                                        READY   STATUS      RESTARTS      AGE
ingress-nginx-admission-create-l6fgw        0/1     Completed   0             11m
ingress-nginx-admission-patch-sfgc4         0/1     Completed   0             11m
ingress-nginx-controller-5f8964959d-278rc   0/1     Error       2 (26s ago)   29s
```

You would see an Error or CrashLoopBackOff in the ingress-nginx-controller. Inspect the logs of the controller pod.
```
cluster3-controlplane ~ ✖ k logs -n ingress-nginx ingress-nginx-controller-5f8964959d-278rc 
-------------------------------------------------------------------------------
--------
F0316 08:03:28.111614      57 main.go:83] No service with name default-backend-service found in namespace default:

-------
```

You see an error msg saying "No service with name default-backend-service found in namespace default".

- We don't have the service with that name in the default namespace, so we need to edit the ingress controller deployment to use the service that we have .i.e. `default-backend-service` in the `green-space` namespace.
  
- To create the controller deployment with correct backend service, first save the deployment in a file, delete the controller deployment, edit the file and create the deployment.

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

- Fix Ingress Resource Path Names
  
Check the current Ingress paths:
```bash
kubectl get ingress ingress-resource-uxz -n green-space -o yaml
```

- You will see paths like /app1 and /app2 pointing to app-wear-service and app-video-service.

Edit the Ingress resource to match your application endpoints:
```bash
kubectl edit ingress ingress-resource-uxz -n green-space
```

Update the paths as follows:
```
paths:
    - backend:
        service:
          name: app-wear-service
          port:
            number: 8080
      path: /app1      # Update this to /app-wear
      pathType: Prefix
    - backend:
        service:
          name: app-video-service
          port:
            number: 8080
      path: /app2        # Update this to /app-video
      pathType: Prefix
```

Save and exit the editor.

- **Verify Ingress Accessibility**
  
After updating, check your endpoints:
```bash
ssh cluster3-controlplane

cluster3-controlplane ~ ➜ curl localhost:30080/app-wear
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #2980b9;">

<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">
    <img src="https://res.cloudinary.com/cloudusthad/image/upload/v1547052428/apparels.jpg">

</div>

</body>

cluster3-controlplane ~ ➜  curl localhost:30080/app-video
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #30336b;">

<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">
    <img src="https://res.cloudinary.com/cloudusthad/image/upload/v1547052431/video.jpg">

</div>

</body>
```

</details>

**Details (Verification):**

* Is the ingress controller issue fixed?
* Is the ingress path /app-wear working?
* Is the ingress path /app-video working?

-----

### Question 11

**Context:** `kubectl config use-context cluster2`

**Task:**
For this scenario, we have already deployed an application in the `global-space`. Inspect them and create an ingress resource with name `ingress-resource-xnz` to make the application available at `/eat` on the Ingress service. Use ingress class of `nginx`.   
  
Also, make use of following annotation fields: -
```
nginx.ingress.kubernetes.io/rewrite-target: /
nginx.ingress.kubernetes.io/ssl-redirect: "false"
```

<details>
<summary><b>View Solution</b></summary>

Switch to cluster2 by using the following command:
```bash
kubectl config use-context cluster2
```

To view the applications running on `global-space` namespace, run the following.
```
cluster2-controlplane ~ ➜  kubectl get pod,svc -n global-space
NAME                                 READY   STATUS    RESTARTS   AGE
pod/default-backend-b46b9989-p9h28   1/1     Running   0          95s
pod/webapp-food-dcd846f95-khhmw      1/1     Running   0          95s

NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/default-backend-service   ClusterIP   10.102.22.145   <none>        80/TCP     95s
service/food-service              ClusterIP   10.99.255.77    <none>        8080/TCP   95s
```

We have service `food-service` configured, this will act as backend service for `/eat` path respectively.

- **Using Command Line**
  
Create an ingress resource using the following imperative command:
```bash
kubectl create ingress ingress-resource-xnz \
  --namespace global-space \
  --rule='/eat'='food-service:8080' \
  --annotation='nginx.ingress.kubernetes.io/rewrite-target=/' \
  --annotation='nginx.ingress.kubernetes.io/ssl-redirect=false' \
  --class=nginx
```

- **Using manifest file**
  
Solution manifest file to create a `ingress` resource as follows:
```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-xnz
  namespace: global-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /eat
        pathType: Prefix
        backend:
          service:
           name: food-service
           port:
            number: 8080
```

</details>

**Details (Verification):**

* Is ingress resource created?
* Does path /eat select `food-service`?
* Is backend port for /food service correct?

-----

### Question 12

**Context:** `kubectl config use-context cluster1`

**Task:**
Create an nginx pod called `nginx-resolver-ckad03-svcn` using image `nginx`, and expose it internally at port `80` with a service called `nginx-resolver-service-ckad03-svcn`.

<details>
<summary><b>View Solution</b></summary>

Switching to `cluster1`:
```bash
kubectl config use-context cluster1
```

To create a pod `nginx-resolver-ckad03-svcn` and expose it internally:
```bash
student-node ~ ➜ kubectl run nginx-resolver-ckad03-svcn --image=nginx 
student-node ~ ➜ kubectl expose pod/nginx-resolver-ckad03-svcn --name=nginx-resolver-service-ckad03-svcn --port=80 --target-port=80 --type=ClusterIP 
```

</details>

**Details (Verification):**

* Is pod: `nginx-resolver-ckad03-svcn` created
* Is pod "nginx-resolver-ckad03-svcn" is exposed using "nginx-resolver-service-ckad03-svcn" ?

-----

### Question 13

**Context:** `kubectl config use-context cluster2`

**Task:**
Create a Deployment named `ckad13-deployment` with "two replicas" of `nginx` image and expose it using a service named `ckad13-service`.   
  
  
Please be noted that service needs to be accessed from both inside and outside the cluster (use port `31080`).

<details>
<summary><b>View Solution</b></summary>

- The following manifest can be used to create an deployment `ckad13-deployment` with `nginx` image and `2` replicas.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ckad13-deployment
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

- To access from outside the cluster, we use nodeport type of service.   
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ckad13-service
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 31080
```

</details>

**Details (Verification):**

* Is deployment `ckad13-deployment` created?
* Is service `ckad13-service` created?
* Is service accessible from outside the cluster?

-----


## Application Environment,  Configuration And Security

### Question 14

**Context:** `kubectl config use-context cluster1`

**Task:**
We have a sample CRD at `/root/ckad10-crd-aecs.yaml` which should have the following validations:

- `destinationName`, `country`, and `city` must be **string** types.
  
- `pricePerNight` must be an **integer** between `50` and `5000`.
  
- `durationInDays` must be an **integer** between `1` and `30`.

Update the file incorporating the above validations in a `namespaced` scope.

`Note`: Remember to create the CRD after the required changes.

<details>
<summary><b>View Solution</b></summary>
```yaml
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  vim ckad10-crd-aecs.yaml 

student-node ~ ➜  cat ckad10-crd-aecs.yaml 
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

student-node ~ ➜  k create -f ckad10-crd-aecs.yaml
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

### Question 15

**Context:** `kubectl config use-context cluster3`

**Task:**
Create a `ClusterRole` named `healthz-access` that allows `GET` and `POST` **requests** to the **non-resource** endpoint `/healthz` and all `subpaths`.  
  
  
Bound this **ClusterRole** to a user `healthz-user` using a **ClusterRoleBinding** named `healthz-access-binding`.

<details>
<summary><b>View Solution</b></summary>
```
student-node ~ ➜  kubectl config use-context cluster3
Switched to context "cluster3".

student-node ~ ➜  kubectl create clusterrole healthz-access --non-resource-url=/healthz,/healthz/* --verb=get,post
clusterrole.rbac.authorization.k8s.io/healthz-access created


student-node ~ ➜  kubectl create clusterrolebinding healthz-access-binding --clusterrole=healthz-access --user=healthz-user
clusterrolebinding.rbac.authorization.k8s.io/healthz-access-binding created
```

</details>

**Details (Verification):**

* Are correct verbs specified?
* Are correct urls specified?
* Is correct clusterrole used for ClusterRoleBinding?
* Is correct user specified for ClusterRoleBinding?

-----

### Question 16

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a ConfigMap named `ckad03-config-aecs` in the `default` namespace with below specifications:  
  
  
**Name**: `ckad03-config-aecs`

* key1=`Exam`, value1=`utlimate-mock-ckad`
* key2=`Provider`, value2=`kodekloud`

<details>
<summary><b>View Solution</b></summary>
```yaml
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  kubectl create configmap ckad03-config-aecs --from-literal=Exam=utlimate-mock-ckad --from-literal=Provider=kodekloud
configmap/ckad03-config-aecs created

student-node ~ ➜  k get cm 
ckad03-config-aecs  kube-root-ca.crt    

student-node ~ ➜  k get cm ckad03-config-aecs -o yaml
apiVersion: v1
data:
  Exam: utlimate-mock-ckad
  Provider: kodekloud
kind: ConfigMap
metadata:
  name: ckad03-config-aecs
  namespace: default
```

</details>

**Details (Verification):**

* Is the correct value for key 'Exam' configured ?
* Is the correct value for key 'Provider' configured ?

-----

### Question 17

**Context:** `kubectl config use-context cluster1`

**Task:**
Define a Kubernetes custom resource definition (CRD) for a new resource **kind** called `Foo` (plural form - `foos`) in the `samplecontroller.example.com` group.  
  
This CRD should have a version of `v1alpha1` with a schema that includes two properties as given below:

> `deploymentName` (a string type) and `replicas` (an integer type with minimum value of 1 and maximum value of 5).

It should also include a `status` **subresource** which enables retrieving and updating the status of **Foo** object, including the `availableReplicas` property, which is an `integer` type.   
The **Foo** resource should be `namespace` **scoped**.  
  
  
`Note`: We have provided a template `/root/foo-crd-aecs.yaml` for your ease. There are few issues with it so please make sure to incorporate the above requirements before deploying on cluster.

<details>
<summary><b>View Solution</b></summary>
```yaml
student-node ~ ➜  kubectl config use-context cluster1
Switched to context "cluster1".

student-node ~ ➜  vim foo-crd-aecs.yaml

student-node ~ ➜  cat foo-crd-aecs.yaml 
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

student-node ~ ➜  kubectl apply -f foo-crd-aecs.yaml
customresourcedefinition.apiextensions.k8s.io/foos.samplecontroller.example.com created
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


## Application Observability And Maintenance

### Question 18

**Context:** `kubectl config use-context cluster1`

**Task:**
Create a new pod with image `nginx` and name `ckad-probe-aom` and configure the pod with `livenessProbe` with `command` `ls` and set `initialDelaySeconds` to 5 .

<details>
<summary><b>View Solution</b></summary>

Using imperative command
```bash
kubectl run ckad-probe-aom --image=nginx  --dry-run=client -o yaml > ckad-probe-aom.yaml
```

Use the following YAML file update yaml with livenessProbe
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: ckad-probe-aom
spec:
  containers:
    - image: nginx
      imagePullPolicy: IfNotPresent
      name: nginx
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

`kubectl create -f ckad-probe-aom.yaml`

</details>

**Details (Verification):**

* Is pod `ckad-probe-aom` running?
* Is nginx image used for the pod?
* is livenessProbe configured?

-----

### Question 19

**Context:** `kubectl config use-context cluster1`

**Task:**
A pod called `kodekloud-logs-aom` is running in the default namespace. It has two co-located containers; get the logs of the `sidecar` container and copy them to `/root/ckad-exam.aom` on student-node.

<details>
<summary><b>View Solution</b></summary>

Identify the containers name in **kodekloud-logs-aom** with
```bash
kubectl get pods kodekloud-logs-aom -o json | jq '.spec.containers[].name'
"ckad-exam"
"sidecar"
```

Use following command to logs of sidecar container
```bash
kubectl logs kodekloud-logs-aom -c sidecar > /root/ckad-exam.aom
```

</details>

**Details (Verification):**

* Logs present

-----

### Question 20

**Context:** `kubectl config use-context cluster1`

**Task:**
Three pods `hulk,thor and ironman` were created on `cluster1`. Of the three pods, identify the following and copy them to below file,

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
kubectl get pod hulk -o json | jq -r '.spec.containers[].resources.limits.memory'
```

Write the content to`/root/pod-metrics` file.

</details>

**Details (Verification):**

* Metrics logs
* Resourcelimit

-----

### Question 21

**Context:** `kubectl config use-context cluster1`

**Task:**
Update the newly created pod `analytics-app` with a `readinessProbe` using the given specifications.   
  
Configure an `HTTP` readiness probe to the existing pod simple-webapp with `path` value set to `/ready` and `port` number to access container is `8080`.

<details>
<summary><b>View Solution</b></summary>

Use the following YAML file and create a file - for example, `analytics-app.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-08-01T04:55:35Z"
  labels:
    name: simple-webapp
  name: analytics-app
  namespace: default
spec:
  containers:
  - env:
    - name: APP_START_DELAY
      value: "80"
    image: kodekloud/webapp-delayed-start
    imagePullPolicy: Always
    name: simple-webapp
    ports:
    - containerPort: 8080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
```

To recreate the pod, run the command:  
  
`kubectl replace -f analytics-app.yaml --force`

</details>

**Details (Verification):**

* Is pod `analytics-app` running?
* Is the image name: kodekloud/webapp-delayed-start
* Is the Readiness Probe: httpGet
* Is the "Http Probe: /ready"
* Is the ReadinessProbe port to 8080

-----
