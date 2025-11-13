---
title: Class 1
parent: 'Module 12 k8s-1'
layout: page
nav_order: 1
---

### Container Orchestration

we can run our application with docker in a single instance but in a production environment we need to scale up and scale down number of instances according to read time traffic. Also when an application is installed in a single instance, it creates single point of failure. So we need to scale it. There are several techniques that facilitates container clustering. 

1. docker swarm
2. kubernetes

---

### Minikube

We can spin up a kubernetes cluster in the cloud. But that's for production. To test and work with `kubernetes` for practice, we can spin up a very basic kubernetes cluster here in our laptop. We will use `minikube` to practice and understand `kubernetes` basics.

### Install Minikube

1. First Install `kubectl`

    If kubectl is not installed, minikube will face difficulty to configure kube config. The following command installs `kubectl` on Linux AMD 64.

    ```shell
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    ```

    Detail about other installation procedure can be found [here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).

2. No we can install `minikube`

    N:B: Commands are for linux amd64.

    ```shell
    curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
    ```

3. Start minikube

    ```shell
    minikube start
    ```

4. Explore minikube commands

    ```shell
    minikube pause
    minikube unpause
    minikube stop
    minikube addons list
    ```

    For more, explore [minikube Get Started blog](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download).

### kubectl

`kubectl` is the cli client for kubernetes. We can interact with kubernetes cluster with this client. When we started minikube cluster, it stored a kubernetes configuration in `.kubec/config` file. Currently, we only have minikube cluster. But in a real scenario, most of the cases, we might need to interact with multiple clusters. In Kubernetes, a kubectl context is a named collection of access parameters that define which Kubernetes cluster you are interacting with, the user credentials to use for authentication, and the default namespace for commands. Contexts are stored in kubeconfig file, typically located at ~/.kube/config. We currently have single k8s cluster configured, so don't need to mention which context to use in our kubectl commands. But later we will need to use it.

Let's try to interact with our k8s cluster and understand various k8s component.

### namespace

Let's run 

```shell
kubectl get namespace
```
It will list down all the namespaces in the cluster.

What's a `namespace`? According to [k8s official documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

> In Kubernetes, namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc.) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc.).

If we further look into the result of `kubectl get namespace`, we will see the following namespaces.

```shell
rt@bp:~$ kubectl get namespaces
NAME                   STATUS   AGE
default                Active   3d1h
kube-node-lease        Active   3d1h
kube-public            Active   3d1h
kube-system            Active   3d1h
kubernetes-dashboard   Active   12m
```

Let's check what's inside each namespace. 

To list down all objects inside `default` namespace, we need to run the following command.

```shell
rt@bp:~$ kubectl get all -n default
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d1h
```
Here, `-n` is used to specify namespace. Well, as the name suggests, `default` namespace is the default namespace. If we don't mention any namespace with `-n`, `default` namespace will be used by default.

```shell
rt@bp:~$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d1h
```

Let's explore other namespaces ... 

```shell
rt@bp:~$ kubectl get all kube-node-lease
error: you must specify only one resource
rt@bp:~$ kubectl get all -n kube-node-lease
No resources found in kube-node-lease namespace.
rt@bp:~$ kubectl get all -n kube-public
No resources found in kube-public namespace.
rt@bp:~$ kubectl get all -n kube-system
NAME                                   READY   STATUS    RESTARTS        AGE
pod/coredns-66bc5c9577-zvfwj           1/1     Running   2 (2d20h ago)   3d1h
pod/etcd-minikube                      1/1     Running   2 (2d20h ago)   3d1h
pod/kube-apiserver-minikube            1/1     Running   2 (26m ago)     3d1h
pod/kube-controller-manager-minikube   1/1     Running   2 (2d20h ago)   3d1h
pod/kube-proxy-dr48r                   1/1     Running   2 (2d20h ago)   3d1h
pod/kube-scheduler-minikube            1/1     Running   2 (2d20h ago)   3d1h
pod/storage-provisioner                1/1     Running   4 (26m ago)     3d1h

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   3d1h

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   3d1h

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           3d1h

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-66bc5c9577   1         1         1       3d1h
```

Yap things are running inside `kube-system` namespace. Let's get familar with other kubernetes objects.

Let's create a new namespace.

```shell
kubectl create namespace ostad
```

With the command we have created a namespace named `ostad`. From now on, in this class we will use `ostad` namespace.

### Pod

A pod is the smallest kubernetes object that can be replicated. In most cases a `pod` consists of a single `container`. But it is possible for a pod to have multiple containers, volumes etc. [More about pod](https://kubernetes.io/docs/concepts/workloads/pods/).

Let's run a pod in `ostad` namespace. 

```shell
kubectl run nginx --image=nginx -n ostad
```
This will spin up a single pod with single container inside. We can check it with the following command. 

```shell
rt@bp:~$ kubectl get pod/nginx -n ostad
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          9s
```

While `kubectl get <object> <name>` gives brief information about k8s object, `kubectl describe` shows descriptive information.

```shell
rt@bp:~$ kubectl describe pod/nginx -n ostad
Name:             nginx
Namespace:        ostad
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Thu, 23 Oct 2025 09:31:52 +0600
Labels:           run=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.0.80
IPs:
  IP:  10.244.0.80
Containers:
  nginx:
    Container ID:   docker://be2600b9aeed24849e738cbfd953dcf2e3ff139508aad906a147024e07e252f6
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:029d4461bd98f124e531380505ceea2072418fdf28752aa73b7b273ba3048903
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 23 Oct 2025 09:31:56 +0600
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5crjh (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-5crjh:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  60s   default-scheduler  Successfully assigned ostad/nginx to minikube
  Normal  Pulling    60s   kubelet            Pulling image "nginx"
  Normal  Pulled     57s   kubelet            Successfully pulled image "nginx" in 3.274s (3.274s including waiting). Image size: 151840157 bytes.
  Normal  Created    57s   kubelet            Created container: nginx
  Normal  Started    57s   kubelet            Started container nginx
```

From pod description, we can check 

1. The pod has got an uniq IP Address.
2. We can use `kubectl describe` command to check the steps taken by k8s cluster to run this pod.
3. Sometimes, a pod might fail to spin up and show error. On that time we can further analyze the problem using `kubectl describe` command.

Another useful kubectl command is `kubectl logs`. We can see logs from `pod` using it. 

```shell
rt@bp:~$ kubectl logs -f pod/nginx -n ostad
```
Ok. The pod is running. It has got uniq ip. But it's inside the cluster. There's kubectl command that we can use to make a bridge between our local machine to the pod using `kubectl port-forwarding`. 

```shell
kubectl port-forward pod/nginx 8080:80 -n ostad
```

### Deployment

deployment: We don't just run pods right? Sometimes we need to scale it up, scale it down, update it, downgrade it, allocate resource on it etc. `deployment` does the task. A Deployment provides declarative updates for Pods and ReplicaSets.

Before 

```shell
kubectl create deployment nginx --
```



### Replica Set

ReplicaSet: According the kubernetes official documentation, A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. Usually, you define a Deployment and let that Deployment manage ReplicaSets automatically.

### Service

Service: Expose an application running in your cluster behind a single outward-facing endpoint, even when the workload is split across multiple backends.

We don't need to know about daemonset now, we will explore `daemonset` and other kubernetes objects later. But for the very curious minds, you can exlore [here](https://kubernetes.io/docs/concepts/workloads/controllers/).



ToDo:

```shell
rt@bp:~$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```


### Servcie 

```shell
kubectl delete pod nginx
kubectl run nginx --image=nginx --expose --port 80
kubectl expose pod/nginx --port=<> --target-port=80
```

Or with deployment 

```shell
kubectl create deployment nginx --image=nginx --replicas=2
kubectl expose deployment/nginx --port=8080 --target-port=80
```

We will see, a service is now created to expose port 80. 

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_expose/ 


```shell
kubectl get all
```

There are 4 types of services in kubernetes 

1. ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a type for a Service. You can expose the Service to the public internet using an Ingress or a Gateway.
2. NodePort: Exposes the Service on each Node's IP at a static port (the NodePort). To make the node port available, Kubernetes sets up a cluster IP address, the same as if you had requested a Service of type: ClusterIP.
3. LoadBalancer:  the Service externally using an external load balancer. Kubernetes does not directly offer a load balancing component; you must provide one, or you can integrate your Kubernetes cluster with a cloud provider.
4. ExternalName: Maps the Service to the contents of the externalName field (for example, to the hostname api.foo.bar.example). The mapping configures your cluster's DNS server to return a CNAME record with that external hostname value. No proxying of any kind is set up.

- https://kubernetes.io/docs/reference/kubectl/generated/kubectl_create/kubectl_create_service_externalname/

### ReplicaSet 


Deployment resource makes it easier for updating your pods to a newer version.

Let's say you use ReplicaSet-A for controlling your pods, then You wish to update your pods to a newer version, now you should create Replicaset-B, scale down ReplicaSet-A and scale up ReplicaSet-B by one step repeatedly (This process is known as rolling update). Although this does the job, but it's not a good practice and it's better to let K8S do the job.

A Deployment resource does this automatically without any human interaction and increases the abstraction by one level.

Note: Deployment doesn't interact with pods directly, it just does rolling update using ReplicaSets.

https://stackoverflow.com/questions/69448131/kubernetes-whats-the-difference-between-deployment-and-replica-set

### Ingress: 

https://kubernetes.io/docs/concepts/services-networking/ingress/


