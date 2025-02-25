---
title: "Require Pod priorityClassName"
category: Multi-Tenancy, EKS Best Practices
version: 
subject: Pod
policyType: "validate"
description: >
    A Pod may optionally specify a priorityClassName which indicates the scheduling priority relative to others. This requires creation of a PriorityClass object in advance. With this created, a Pod may set this field to that value. In a multi-tenant environment, it is often desired to require this priorityClassName be set to make certain tenant scheduling guarantees. This policy requires that a Pod defines the priorityClassName field with some value.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/require-pod-priorityclassname/require-pod-priorityclassname.yaml" target="-blank">/other/require-pod-priorityclassname/require-pod-priorityclassname.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-pod-priorityclassname
  annotations:
    policies.kyverno.io/title: Require Pod priorityClassName
    policies.kyverno.io/category: Multi-Tenancy, EKS Best Practices
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      A Pod may optionally specify a priorityClassName which indicates the scheduling
      priority relative to others. This requires creation of a PriorityClass object in advance.
      With this created, a Pod may set this field to that value. In a multi-tenant environment,
      it is often desired to require this priorityClassName be set to make certain tenant
      scheduling guarantees. This policy requires that a Pod defines the priorityClassName field
      with some value.
spec:
  validationFailureAction: Enforce
  background: true
  rules:
  - name: check-priorityclassname
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Pods must define the priorityClassName field."
      pattern:
        spec:
          priorityClassName: "?*"

```
