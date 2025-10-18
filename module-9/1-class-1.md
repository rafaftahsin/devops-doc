---
title: Class 1
parent: 'Module 9'
layout: page
nav_order: 1
---

# Container Orchestration

we can run our application with docker in a single instance but in a production environment we need to scale up and scale down number of instances according to read time traffic. Also when an application is installed in a single instance, it creates single point of failure. So we need to scale it. There are several techniques that facilitates container clustering. 

1. docker swarm
2. kubernetes

# Minikube

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

### `kubectl`

1. `kubectl get pods`

It will list of the pods in the default `namespace`

What's a `namespace`? It's an isolation inside k8s cluster. There are various kinds of objects in k8s cluster. For example pod, deployment, service, ingress, replicaset and a lot other things ... everything runs inside a `namespace`. There are multiple namespaces in a kubernetes cluster. By default, in most cases, things inside a namespace are not disconverable from other namespace. But obviously, you can tweak things. You can list namespaces inside a cluster with `kubectl get namespaces`.

For example, our minikube cluster has the following namespaces.

```shell
rt@bp:~$ kubectl get namespaces
NAME                   STATUS   AGE
default                Active   3d1h
kube-node-lease        Active   3d1h
kube-public            Active   3d1h
kube-system            Active   3d1h
kubernetes-dashboard   Active   12m
```

Let's see what's inside these namespaces. 

To list down all objects inside `default` namespace, we need to run the following command.

```shell
rt@bp:~$ kubectl get all -n default
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d1h
```
Here, `-n` is used to specify namespace. Well, as the name suggests, `default` namespace is the default namespace. If you don't mention any namespace with `n`, `default` namespace will be used by default.

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

Yap things are running inside `kube-system` namespace. But what are those?

pod: A pod is the smallest kubernetes object that can be replicated. In most cases a `pod` consists of a single `container`. But it is possible for a pod to have multiple containers, volumes etc. [More about pod](https://kubernetes.io/docs/concepts/workloads/pods/).

deployment: We don't just run pods right? Sometimes we need to scale it up, scale it down, update it, downgrade it, allocate resource on it etc. `deployment` does the task. A Deployment provides declarative updates for Pods and ReplicaSets.

ReplicaSet: According the kubernetes official documentation, A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. Usually, you define a Deployment and let that Deployment manage ReplicaSets automatically.

Service: Expose an application running in your cluster behind a single outward-facing endpoint, even when the workload is split across multiple backends.

We don't need to know about daemonset now, we will explore `daemonset` and other kubernetes objects later. But for the very curious minds, you can exlore [here](https://kubernetes.io/docs/concepts/workloads/controllers/).

### Let's run job in our default namespace

```shell
kubectl run nginx --image=nginx
```

```shell
kubectl get all # $ kubectl get all -n default
kubectl describe pod nginx
kubectl describe pod/nginx
```

More about kubectl run -> https://kubernetes.io/docs/reference/kubectl/generated/kubectl_run/

We can check nginx is running with kubectl port-forwarding 

```shell
kubectl port-forward pod/nginx 8080:80
```









### Servcie 

```shell
kubectl delete pod nginx
kubectl run nginx --image=nginx --expose --port 80
```

We will see, a service is now created to expose port 80. 

```shell
kubectl get all
```

There are 4 types of services in kubernetes 

1. ClusterIP: This one is the default. Simply a service gets an IP from `namespace` IP Pool. We can access it with `kubectl port-forward`


