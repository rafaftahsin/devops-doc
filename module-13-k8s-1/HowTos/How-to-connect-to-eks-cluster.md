---
title: How to connect to EKS cluster
parent:  'HowTos'
layout: home
grand_parent: 'Module 13 k8s-1'
---

### Connect to cluster

```shell
aws eks update-kubeconfig --name ostad-eks --region ap-southeast-1
```

Ref: https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html