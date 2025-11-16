---
title: How to install ArgoCD
layout: home
parent: 'notes'
grand_parent: 'Module 14 k8s-2'
---

## How to install argocd

### Step 1: Create argbaod namespace

```shell
kubectl create namespace argocd
```
### Step 2: Install ArgoCD manifests in argocd namespace

```shell
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

k8s will take few minutes to spin up everything. Wait till all pods are in running state.

```text
rt@tp:~$ kubectl get all -n argocd
NAME                                                    READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0                     1/1     Running   0          2m37s
pod/argocd-applicationset-controller-7b6ff755dc-l2dl4   1/1     Running   0          2m37s
pod/argocd-dex-server-584f7d88dc-rk5fd                  1/1     Running   0          2m37s
pod/argocd-notifications-controller-67cdd486c6-nsl7z    1/1     Running   0          2m37s
pod/argocd-redis-6dbb9f6cf4-hgjpz                       1/1     Running   0          2m37s
pod/argocd-repo-server-57bdcb5898-pnfmm                 1/1     Running   0          2m37s
pod/argocd-server-57d9cc9bcf-cgn82                      1/1     Running   0          2m37s

NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP   10.111.89.171   <none>        7000/TCP,8080/TCP            2m37s
service/argocd-dex-server                         ClusterIP   10.106.18.212   <none>        5556/TCP,5557/TCP,5558/TCP   2m37s
service/argocd-metrics                            ClusterIP   10.108.77.62    <none>        8082/TCP                     2m37s
service/argocd-notifications-controller-metrics   ClusterIP   10.103.58.243   <none>        9001/TCP                     2m37s
service/argocd-redis                              ClusterIP   10.108.167.62   <none>        6379/TCP                     2m37s
service/argocd-repo-server                        ClusterIP   10.103.1.80     <none>        8081/TCP,8084/TCP            2m37s
service/argocd-server                             ClusterIP   10.101.42.109   <none>        80/TCP,443/TCP               2m37s
service/argocd-server-metrics                     ClusterIP   10.102.26.206   <none>        8083/TCP                     2m37s

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           2m37s
deployment.apps/argocd-dex-server                  1/1     1            1           2m37s
deployment.apps/argocd-notifications-controller    1/1     1            1           2m37s
deployment.apps/argocd-redis                       1/1     1            1           2m37s
deployment.apps/argocd-repo-server                 1/1     1            1           2m37s
deployment.apps/argocd-server                      1/1     1            1           2m37s

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-7b6ff755dc   1         1         1       2m37s
replicaset.apps/argocd-dex-server-584f7d88dc                  1         1         1       2m37s
replicaset.apps/argocd-notifications-controller-67cdd486c6    1         1         1       2m37s
replicaset.apps/argocd-redis-6dbb9f6cf4                       1         1         1       2m37s
replicaset.apps/argocd-repo-server-57bdcb5898                 1         1         1       2m37s
replicaset.apps/argocd-server-57d9cc9bcf                      1         1         1       2m37s

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     2m37s
```

Now that all pods are running, we can access argocd UI using port-forward.

### Step 3: Access ArgoCD UI

```shell
kubectl port-forward svc/argocd-server -n argocd 8080:80
```
Here, 8080 is our port on our local machine and 80 is the port exposed by the service. If we run `kubectl get svc -n argocd` we can see that the service is exposed on port 80.

Now open browser and go to http://localhost:8080.

### Step 4: Initial Credentials


Initial credentials are:

username: admin
password:

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

- Ref: [Stackoverflow QA](https://stackoverflow.com/questions/68297354/what-is-the-default-password-of-argocd)

### References

- [Installation Documentation](https://argo-cd.readthedocs.io/en/latest/operator-manual/installation/)

### Step 5: (Optional) configuring ingress


There may be a debate on whether argocd should be exposed via ingress or not. Here's an example ingress configuration for argocd using nginx ingress controller.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-http-ingress
  namespace: argocd
  annotations:
    cert-manager.io/issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  name: https
      host: argocd.example.com
  tls:
    - hosts:
        - argocd.example.com
      secretName: argocd-ingress-http
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-prod
  namespace: argocd
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: someone@example.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
      - http01:
          ingress:
            ingressClassName: nginx
```

- Ref: [Official Documentation](https://argo-cd.readthedocs.io/en/latest/operator-manual/ingress/)

