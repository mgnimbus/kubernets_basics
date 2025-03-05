# K8s-101

Reference:

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

https://kubernetes.io/docs/reference/kubectl/conventions/

To set an alias in new env
```
alias k=kubectl
```
## Pod 

- Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

- The "one-container-per-Pod" model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container; Kubernetes manages Pods rather than managing the containers directly.

- Pod can contain init containers that run during Pod startup. You can also inject ephemeral containers for debugging a running Pod

- Pods that run multiple containers that need to work together. A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers form a single cohesive unit.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: myapp-pod
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

## ReplicaSet

- A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

- This actually means that you may never need to manipulate ReplicaSet objects: use a Deployment instead, and define your application in the spec section


```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
  labels:
      type: front-end
spec:
  template:
    metadata:
      labels:
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end              
```

- To incerese the no.of replicas you can edit the deployment file and run it or run the kctl cmd as below. But doest mean we can increase the count to 6

```bash
kubectl scale --replicas=6 replicaset myapp-rs

kubectl get rs 
kubectl delete rs myapp-rs # also delete underlying pods
kubectl replace -f myapp-ra.yaml # update the rs
```

# Deployments

- A Deployment provides declarative updates for Pods and ReplicaSets.

- You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-frontend
  labels:
      type: frontend-deploy
spec:
  template:
    metadata:
      labels:
        type: front-end
    spec:
      containers:
        - name: http-container
          image: httpd:2.4-alpine
  replicas: 3
  selector:
    matchLabels:
      type: front-end
 ```bash
kubectl create -f myapp-deploy.yaml
kubectl scale --replicas=6 replicaset myapp-rs

kubectl get deoply 
kubectl delete deploy myapp-rs # also delete underlying pods
kubectl replace -f myapp-deploy.yaml # update the rs
```

### To Create the above workload with kubectl command


Create an NGINX Pod

```
kubectl run nginx --image=nginx
```
Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```
Create a deployment
```
kubectl create deployment --image=nginx nginx
```
Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```
Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.
```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```
Make necessary changes to the file (for example, adding more replicas) and then create the deployment.
```
kubectl create -f nginx-deployment.yaml
```
In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

```
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml

kubectl create deployment --image=busybox deployment-1 --replicas=4 -o yaml > deployment-definition-1.yaml
```


# service 

Expose an application running in your cluster behind a single outward-facing endpoint, even when the workload is split across multiple backends.

In Kubernetes, a Service is a method for exposing a network application that is running as one or more Pods in your cluster.You use a Service to make that set of Pods available on the network so that clients can interact with it.

If you use a Deployment to run your app, that Deployment can create and destroy Pods dynamically. From one moment to the next, you don't know how many of those Pods are working and healthy; you might not even know what those healthy Pods are named. Kubernetes Pods are created and destroyed to match the desired state of your cluster. Pods are ephemeral resources (you should not expect that an individual Pod is reliable and durable).

Each Pod gets its own IP address (Kubernetes expects network plugins to ensure this). For a given Deployment in your cluster, the set of Pods running in one moment in time could be different from the set of Pods running that application a moment later.

This leads to a problem: if some set of Pods (call them "backends") provides functionality to other Pods (call them "frontends") inside your cluster, how do the frontends find out and keep track of which IP address to connect to, so that the frontend can use the backend part of the workload?


## type: NodePort

For a node port Service, Kubernetes additionally allocates a port (TCP, UDP or SCTP to match the protocol of the Service). Every node in the cluster configures itself to listen on that assigned port and to forward traffic to one of the ready endpoints associated with that Service. You'll be able to contact the type: NodePort Service, from outside the cluster, by connecting to any node using the appropriate protocol (for example: TCP), and the appropriate port (as assigned to that Service)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  labels:
    tier: frnd-svc
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: MyApp
  ports:
      # Exposed in serivce complusory field
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      # where the application is exposed in pod
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```
Simple NodePort svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
  labels:
    tier: myapp-frnd-svc
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30007
  selector:
    name: myapp
    type: frontend

``` 

## type: ClusterIP
Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a type for a Service. You can expose the Service to the public internet using an Ingress or a Gateway.

Simple clusterIP svc

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-bknd-svc
  labels:
    tier: myapp-bknd-svc
spec:
  type: ClusterIP #Default
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30007
  selector:
    name: myapp-db
    type: backend

``` 

## type: LoadBalancer

- Traffic from the external load balancer is directed at the backend Pods.T

- To implement a Service of type: LoadBalancer, Kubernetes typically starts off by making the changes that are equivalent to you **requesting a Service of type: NodePort**. The cloud-controller-manager component then configures the external load balancer to forward traffic to that assigned node port.


Simple LoadBalancer svc
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-svc
spec:
  selector:
    app: myapp
  ports:
  type: LoadBalancer
    - protocol: TCP
      port: 80
      targetPort: 9376
  clusterIP: 10.0.171.239
```

# Namespace

In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping **is applicable only for namespaced objects (e.g. Deployments, Services, etc.)** and **not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc.)**.

## Kubernetes starts with four initial namespaces:

- **default**
    - Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace.
    
- **Kube-node-lease**
    - This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.

- **kube-public**
    - This namespace is readable by all clients (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.

- **kube-system**
    - The namespace for objects created by the Kubernetes system


### Creating a namespace

```
k create ns dev

k get ns

kubectl get namespaces --show-labels

k delete ns test #This deletes everything under the namespace!

k get po -n kube-system

kubectl get pods -l app=myapp -n=dev
```

## Namespaces and DNS
When you create a Service, it creates a corresponding DNS entry. This entry is of the form **service-name.namespace-name.svc.cluster.local**, which means that if a container only uses <service-name>, it will resolve to the service which is local to a namespace. This is useful for using the same configuration across multiple namespaces such as Development, Staging and Production. If you want to reach across namespaces, you need to use the fully qualified domain name (FQDN).

## Not all objects are in a namespace
Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are in some namespaces. However namespace resources are not themselves in a namespace. And low-level resources, such as nodes and persistentVolumes, are not in any namespace

#### In a namespace
```
kubectl api-resources --namespaced=true
```
#### Not in a namespace
```
kubectl api-resources --namespaced=false
```
## Automatic labelling 
The Kubernetes control plane sets an immutable label **kubernetes.io/metadata.name on all namespaces**. The value of the label is the namespace name


To deploy pod in a particular ns

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
  labels:
    app: myapp
    type: frontend
spec:
  containers:
  - name: nginx
    image: nginx
```

To set the dev as context

```
k config set-context $(kubectl config current-context) --namespace=dev
```

## Resource Quotas

A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per namespace. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that namespace.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: dev
spec:
  hard:
    pod: "10"
    requests.cpu: "1"
    requests.memory: "1Gi"
    limits.cpu: "2"
    limits.memory: "2Gi"
```

## Imperative vs Declarative 

### Imperative

### Create Objects

```
k run pod nginx --image=nginx -n dev

k create deploy nginx --image=nginx

```
### Update Objects

```
k scale deply nginx-deploy --replica=5

# Set a deployment's nginx container image to 'nginx:1.9.1'
k set image deploy nginx nginx=nginx:1.9.1

# Depletes and recreates the objects
k replace --force -f values.dev.yaml -n tempo
```
### Declarative

```
k apply -f dev.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nicepod
  labels:
    App: dev
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
```


While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save considerable amount of time during your exams.

Before we begin, familiarize with the two options that can come in handy while working with the below commands:

--dry-run: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the --dry-run=client option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.

-o yaml: This will output the resource definition in YAML format on screen.



Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.


### POD

Create an NGINX Pod
```
kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

### Deployment

Create a deployment
```
kubectl create deployment --image=nginx nginx
```

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

Generate Deployment with 4 Replicas
```
kubectl create deployment nginx --image=nginx --replicas=4
```

You can also scale a deployment using the kubectl scale command.
```
kubectl scale deployment nginx --replicas=4
```
Another way to do this is to save the YAML definition to a file and modify
```
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

You can then update the YAML file with the replicas or any other field before creating the deployment.


### Service

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
```
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```
(This will automatically use the pod's labels as selectors)

Or

kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)


Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
```
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```
(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or
```
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```
(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the kubectl expose command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

