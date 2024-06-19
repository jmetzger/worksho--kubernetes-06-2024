# Restricted only Ingress but all egress possible 

```
---
apiVersion: crd.projectcalico.org/v1
kind: GlobalNetworkPolicy
metadata:
  name: default-deny
spec:
  namespaceSelector: kubernetes.io/metadata.name != "kube-system"
  types:
  - Ingress
---
# 03-allow-ingress-my-nginx.yml
apiVersion: crd.projectcalico.org/v1
kind: NetworkPolicy
metadata:
  name: allow-only-from-within-namespace
  namespace: fromhere
spec:
  types:
  - Ingress
  ingress:
  - action: Allow
    source:
      namespaceSelector: kubernetes.io/metadata.name == "fromhere"
---
# 03-allow-ingress-my-nginx.yml
apiVersion: crd.projectcalico.org/v1
kind: NetworkPolicy
metadata:
  name: allow-only-from-within-namespace
  namespace: np
spec:
  types:
  - Ingress
  ingress:
  - action: Allow
    source:
      namespaceSelector: kubernetes.io/metadata.name == "np"

```
