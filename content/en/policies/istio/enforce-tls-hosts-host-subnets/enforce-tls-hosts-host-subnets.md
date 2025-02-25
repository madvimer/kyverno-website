---
title: "Enforce Istio TLS on Hosts and Host Subnets"
category: Istio
version: 1.6.0
subject: DestinationRule
policyType: "validate"
description: >
    Once a routing decision has been made, a DestinationRule can be used to define how traffic should be sent to another service. The trafficPolicy object can control how TLS is handled to the destination host. This policy enforces that the TLS mode cannot be set to a value of `DISABLE`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//istio/enforce-tls-hosts-host-subnets/enforce-tls-hosts-host-subnets.yaml" target="-blank">/istio/enforce-tls-hosts-host-subnets/enforce-tls-hosts-host-subnets.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: enforce-tls-hosts-host-subnets
  annotations:
    policies.kyverno.io/title: Enforce Istio TLS on Hosts and Host Subnets
    policies.kyverno.io/category: Istio
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: DestinationRule
    kyverno.io/kyverno-version: 1.8.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/description: >- 
      Once a routing decision has been made, a DestinationRule can be used to define how traffic
      should be sent to another service. The trafficPolicy object can control how TLS is handled
      to the destination host. This policy enforces that the TLS mode cannot be set to a value
      of `DISABLE`.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: destrule
    match:
      any:
      - resources:
          kinds:
          - DestinationRule
    validate:
      message: "TLS may not be disabled for the trafficPolicy in any host."
      pattern:
        =(spec):
          =(trafficPolicy):
            =(tls):
              =(mode): "!DISABLE"
```
