---
title: "Restrict Secrets by Name in CEL expressions"
category: Other in CEL
version: 
subject: Pod, Secret
policyType: "validate"
description: >
    Secrets often contain sensitive information and their access should be carefully controlled. Although Kubernetes RBAC can be effective at restricting them in several ways, it lacks the ability to use wildcards in resource names. This policy ensures that only Secrets beginning with the name `safe-` can be consumed by Pods. In order to work effectively, this policy needs to be paired with a separate policy or rule to require `automountServiceAccountToken=false` since this would otherwise result in a Secret being mounted.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other-cel/restrict-secrets-by-name/restrict-secrets-by-name.yaml" target="-blank">/other-cel/restrict-secrets-by-name/restrict-secrets-by-name.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-secrets-by-name
  annotations:
    policies.kyverno.io/title: Restrict Secrets by Name in CEL expressions
    policies.kyverno.io/category: Other in CEL 
    policies.kyverno.io/subject: Pod, Secret
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      Secrets often contain sensitive information and their access should be carefully controlled.
      Although Kubernetes RBAC can be effective at restricting them in several ways,
      it lacks the ability to use wildcards in resource names. This policy ensures
      that only Secrets beginning with the name `safe-` can be consumed by Pods.
      In order to work effectively, this policy needs to be paired with a separate policy
      or rule to require `automountServiceAccountToken=false` since this would otherwise
      result in a Secret being mounted.
spec:
  background: false
  validationFailureAction: Audit
  rules:
  - name: safe-secrets-from-env
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
        variables:
          - name: allContainers
            expression: "object.spec.containers + object.spec.?initContainers.orValue([]) + object.spec.?ephemeralContainers.orValue([])"
        expressions: 
          - expression: >-
              variables.allContainers.all(container, 
              container.?env.orValue([]).all(env,
              env.?valueFrom.?secretKeyRef.?name.orValue('safe-').startsWith("safe-")))
            message: "Only Secrets beginning with `safe-` may be consumed in env statements."
  - name: safe-secrets-from-envfrom
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
        variables:
          - name: allContainers
            expression: "object.spec.containers + object.spec.?initContainers.orValue([]) + object.spec.?ephemeralContainers.orValue([])"
        expressions: 
          - expression: >-
              variables.allContainers.all(container, 
              container.?envFrom.orValue([]).all(env,
              env.?secretRef.?name.orValue('safe-').startsWith("safe-")))
            message: "Only Secrets beginning with `safe-` may be consumed in envFrom statements."
  - name: safe-secrets-from-volumes
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
              object.spec.?volumes.orValue([]).all(volume,
              volume.?secret.?secretName.orValue('safe-').startsWith("safe-"))
            message: "Only Secrets beginning with `safe-` may be consumed in volumes."


```
