---
title: 'Q&A'
parent: 'Module 12 k8s-1'
layout: page
nav_order: 5
---

Question 1: Where the auto scaling happens? Does it happen in deployment or in service?

Answer: It happens in service, not in deployment. When we port-forward a deployment, it forwards traffic from the first pod sorted by name. Traffic distribution is handled by service not deployment.

Question 2: Difference between k8s namespace vs Linux namespace

Answer: https://www.redhat.com/en/blog/kubernetes-namespaces-demystified-how-to-make-the-most-of-them



