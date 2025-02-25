---
title: "Restrict Secrets by Label"
category: Other
version: 1.6.0
subject: Pod, Secret
policyType: "validate"
description: >
    Secrets often contain sensitive information and their access should be carefully controlled. Although Kubernetes RBAC can be effective at restricting them in several ways, it lacks the ability to use labels on referenced entities. This policy ensures that only Secrets not labeled with `status=protected` can be consumed by Pods.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/restrict-secrets-by-label/restrict-secrets-by-label.yaml" target="-blank">/other/restrict-secrets-by-label/restrict-secrets-by-label.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-secrets-by-label
  annotations:
    policies.kyverno.io/title: Restrict Secrets by Label
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kyverno-version: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Pod, Secret
    policies.kyverno.io/description: >-
      Secrets often contain sensitive information and their access should be carefully controlled.
      Although Kubernetes RBAC can be effective at restricting them in several ways,
      it lacks the ability to use labels on referenced entities. This policy ensures
      that only Secrets not labeled with `status=protected` can be consumed by Pods.
spec:
  background: false
  validationFailureAction: Audit
  rules:
  - name: secrets-lookup-from-env
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      # check if any secrets are in env statements
      - key: "{{ request.object.spec.[containers, initContainers, ephemeralContainers][].env[].valueFrom.secretKeyRef || '' | length(@) }}"
        operator: GreaterThanOrEquals
        value: 1
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: NotEquals
        value: DELETE
    validate:
      message: These Secrets are restricted.
      foreach:
      # get every secret, then do an API lookup on each one checking for the `status` label.
      # deny the request if any of those secrets have `status=protected`.
      - list: "request.object.spec.[containers, initContainers, ephemeralContainers][].env[].valueFrom.secretKeyRef"
        context:
        - name: status
          apiCall:
            jmesPath: "metadata.labels.status || ''"
            urlPath: "/api/v1/namespaces/{{request.namespace}}/secrets/{{element.name}}"
        deny:
          conditions:
            any:
            - key: "{{ status }}"
              operator: Equals
              value: protected
  - name: secrets-lookup-from-envfrom
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      # check if any secrets are in envfrom statements
      - key: "{{ request.object.spec.[containers, initContainers, ephemeralContainers][].envFrom[].secretRef || '' | length(@) }}"
        operator: GreaterThanOrEquals
        value: 1
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: NotEquals
        value: DELETE
    validate:
      message: These Secrets are restricted.
      foreach:
      # get every secret, then do an API lookup on each one checking for the `status` label.
      # deny the request if any of those secrets have `status=protected`.
      - list: "request.object.spec.[containers, initContainers, ephemeralContainers][].envFrom[].secretRef"
        context:
        - name: status
          apiCall:
            jmesPath: "metadata.labels.status || ''"
            urlPath: "/api/v1/namespaces/{{request.namespace}}/secrets/{{element.name}}"
        deny:
          conditions:
            any:
            - key: "{{ status }}"
              operator: AnyIn
              value: protected
  - name: secrets-lookup-from-volumes
    match:
      any:
      - resources:
          kinds:
          - Pod
    preconditions:
      all:
      # check if any secrets are in volume statements
      - key: "{{ request.object.spec.volumes[].secret || '' | length(@) }}"
        operator: GreaterThanOrEquals
        value: 1
      - key: "{{request.operation || 'BACKGROUND'}}"
        operator: NotEquals
        value: DELETE
    validate:
      message: These Secrets are restricted.
      foreach:
      # get every secret, then do an API lookup on each one checking for the `status` label.
      # deny the request if any of those secrets have `status=protected`.
      - list: "request.object.spec.volumes[].secret"
        context:
        - name: status
          apiCall:
            jmesPath: "metadata.labels.status || ''"
            urlPath: "/api/v1/namespaces/{{request.namespace}}/secrets/{{element.secretName}}"
        deny:
          conditions:
            any:
            - key: "{{ status }}"
              operator: Equals
              value: protected
```
