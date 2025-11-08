---
title: How to install ArgoCD
---

### How to install argocd

- https://argo-cd.readthedocs.io/en/latest/operator-manual/installation/

### configuring ingress

- https://argo-cd.readthedocs.io/en/latest/operator-manual/ingress/

### argocd default username pass

username: admin
password:

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Ref: https://stackoverflow.com/questions/68297354/what-is-the-default-password-of-argocd