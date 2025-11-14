---
title: How to install EKS cluster with eksctl
parent:  'HowTos'
layout: home
grand_parent: 'Module 13 k8s-1'
---

### Automode

*** Not Done Yet ***

```yaml
# cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: ostad-eks
  region: ap-southeast-1

iam:
  # ARN of the Cluster IAM Role
  # optional, eksctl creates a new role if not supplied
  # suggested to use one Cluster IAM Role per account
  serviceRoleARN: 

autoModeConfig:
  # defaults to false
  enabled: boolean
  # optional, defaults to [general-purpose, system].
  # suggested to leave unspecified
  # To disable creation of nodePools, set it to the empty array ([]).
  nodePools: []
  # optional, eksctl creates a new role if this is not supplied
  # and nodePools are present.
  nodeRoleARN: string
```

Save the ClusterConfig file as `cluster.yaml`. ush the command:

```bash
eksctl create cluster -f cluster.yaml
```

- https://docs.aws.amazon.com/eks/latest/userguide/automode-get-started-eksctl.html

### Manual mode

Ref: eksctl create cluster --name my-cluster --region region-code