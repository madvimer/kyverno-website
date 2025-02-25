---
title: "Restrict control plane scheduling in CEL expressions"
category: Sample in CEL
version: 1.11.0
subject: Pod
policyType: "validate"
description: >
    Scheduling non-system Pods to control plane nodes (which run kubelet) is often undesirable because it takes away resources from the control plane components and can represent a possible security threat vector. This policy prevents users from setting a toleration in a Pod spec which allows running on control plane nodes with the taint key `node-role.kubernetes.io/master`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/restrict-controlplane-scheduling/restrict-controlplane-scheduling.yaml" target="-blank">/other-cel/restrict-controlplane-scheduling/restrict-controlplane-scheduling.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-controlplane-scheduling
  annotations:
    policies.kyverno.io/title: Restrict control plane scheduling in CEL expressions
    policies.kyverno.io/category: Sample in CEL 
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Scheduling non-system Pods to control plane nodes (which run kubelet) is often undesirable
      because it takes away resources from the control plane components and can represent
      a possible security threat vector. This policy prevents users from setting a toleration
      in a Pod spec which allows running on control plane nodes
      with the taint key `node-role.kubernetes.io/master`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: restrict-controlplane-scheduling-master
    match:
      any:
      - resources:
          kinds:
          - Pod
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: >-
              !has(object.spec.tolerations) || 
              !object.spec.tolerations.exists(toleration, toleration.?key.orValue('') in ['node-role.kubernetes.io/master', 'node-role.kubernetes.io/control-plane'])
            message: Pods may not use tolerations which schedule on control plane nodes.


```
