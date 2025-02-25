---
title: "Validate Schedule in CEL expressions"
category: Velero in CEL
version: 
subject: Schedule
policyType: "validate"
description: >
    A Velero Schedule is given in Cron format and must be accurate to ensure operation. This policy validates that the schedule is a valid Cron format.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//velero-cel/validate-cron-schedule/validate-cron-schedule.yaml" target="-blank">/velero-cel/validate-cron-schedule/validate-cron-schedule.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: validate-cron-schedule
  annotations:
    policies.kyverno.io/title: Validate Schedule in CEL expressions
    policies.kyverno.io/category: Velero in CEL 
    policies.kyverno.io/subject: Schedule
    kyverno.io/kyverno-version: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/description: >-
      A Velero Schedule is given in Cron format and must be accurate to ensure
      operation. This policy validates that the schedule is a valid Cron format.
spec:
  background: true
  validationFailureAction: Audit
  rules:
  - name: validate-cron
    match:
      any:
      - resources:
          kinds:
          - velero.io/v1/Schedule
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: >-
              object.spec.schedule.matches('^((?:\\*|[0-5]?[0-9](?:(?:-[0-5]?[0-9])|(?:,[0-5]?[0-9])+)?)(?:\\/[0-9]+)?)\\s+((?:\\*|(?:1?[0-9]|2[0-3])(?:(?:-(?:1?[0-9]|2[0-3]))|(?:,(?:1?[0-9]|2[0-3]))+)?)(?:\\/[0-9]+)?)\\s+((?:\\*|(?:[1-9]|[1-2][0-9]|3[0-1])(?:(?:-(?:[1-9]|[1-2][0-9]|3[0-1]))|(?:,(?:[1-9]|[1-2][0-9]|3[0-1]))+)?)(?:\\/[0-9]+)?)\\s+((?:\\*|(?:[1-9]|1[0-2])(?:(?:-(?:[1-9]|1[0-2]))|(?:,(?:[1-9]|1[0-2]))+)?)(?:\\/[0-9]+)?)\\s+((?:\\*|[0-7](?:-[0-7]|(?:,[0-7])+)?)(?:\\/[0-9]+)?)$')
            message: The backup schedule must be in a valid cron format.


```
