# Istio Authorization with IAP

## Architecture
```
ingress --> istio-ingressgateway --> myapp
```

- ingress has IAP turned on
- ingress and istio-ingressgateway are in istio-system namespace, myapp is in lunkai namespace
- applied policy to validate jwt from IAP at istio-ingressgateway
- used a VirtualService to route to myapp
- myapp has istio sidecar injected (pod has two containers, istio-proxy and myapp)

## Authorization

### Turn on Istio authorization

Create a RBACConfig:
```
apiVersion: "rbac.istio.io/v1alpha1"
kind: RbacConfig
metadata:
  name: default
spec:
  mode: "ON_WITH_INCLUSION"
  inclusion:
    namespaces: ["lunkai"]
```

After this RBACConfig is created, I can't access myapp now

### Grant access using ServiceRole and ServiceRoleBinding
```
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRole
metadata:
  name: service-admin
  namespace: lunkai
spec:
  rules:
  - services: ["*"]
---
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRoleBinding
metadata:
  name: test-binding-products
  namespace: lunkai
spec:
  subjects:
  - properties:
      request.headers[x-goog-authenticated-user-email]: "accounts.google.com:abc@gmail.com"
  - properties:
      request.auth.claims[email]: "abc@gmail.com"
  roleRef:
    kind: ServiceRole
    name: "service-admin"
```

### Key findings
1. Troubleshooting [guide](https://preliminary.istio.io/help/ops/security/debugging-authorization/)
1. The service spec of myapp must have port named http-XX so that Istio recognizes it's http, not tcp.
   This is documented [here](https://istio.io/help/faq/traffic-management/#naming-port-convention).
   Without this port naming, I saw error in Pilot:
   ```
   rules[0] ignored, found HTTP only rule for a TCP service: methods([*]) not supported
   ```
1. With logging for rbac of istio-proxy turned on to `debug` level, we can see the request detail.
   - some headers are available, e.g. `'x-goog-authenticated-user-email', 'accounts.google.com:abc@gmail.com'`
   - however, no request.auth.XX [properties](https://istio.io/docs/reference/config/authorization/constraints-and-properties/#properties)
1. If we apply a same JWT validation policy on this service, then request.auth.X are available.
   `request.auth.claims[email]=abc@gmail.com`
1. When not authorized, we will see `upstream connect error or disconnect/reset before headers` if it's a TCP service. Confusing.


