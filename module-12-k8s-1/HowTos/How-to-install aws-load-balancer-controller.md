---
title: How to install aws-load-balancer-controller
parent:  'HowTos'
layout: home
grand_parent: 'Module 12 k8s-1'
---


```shell
# open ID connector
eksctl utils associate-iam-oidc-provider \
    --region ap-southeast-1 \
    --cluster confused-blues-mongoose \
    --approve

# create IAM policy
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.14.1/docs/install/iam_policy.json
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

# create k8s service account
eksctl create iamserviceaccount \
--cluster=<cluster-name> \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region <region-code> \
--approve

# install helm chart
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    --set clusterName=<cluster-name> \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=<region-code> \
    --set vpcId=<vpc-id>
```

Ref: 
- https://github.com/kubernetes-sigs/aws-load-balancer-controller/issues/3695
- https://github.com/aws/eks-charts/tree/master/stable/aws-load-balancer-controller