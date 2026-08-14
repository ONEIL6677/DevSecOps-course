# 4. Kyverno (Policy as Code)

## What is Kyverno?

Kyverno is a **Kubernetes-native policy engine**.

It can:

* Validate manifests
* Mutate resources
* Generate resources

No new language. Uses YAML.

---

## Install Kyverno

```
kubectl apply --server-side -f https://github.com/kyverno/kyverno/releases/latest/download/install.yaml
```

Verify:

```
kubectl get pods -n kyverno
```

---

## Hands-On: Enforce No Latest Tag

Policy:

```
kubectl apply -f - <<EOF
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-latest-tag
spec:
  validationFailureAction: Enforce
  rules:
  - name: require-image-tag
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Image tag 'latest' is not allowed"
      pattern:
        spec:
          containers:
          - image: "!*:latest"
EOF
```

---

## Test Kyverno Policy

This should fail:

```
kubectl run bad-pod --image=nginx:latest -n payments
```

This should succeed:

```
kubectl run good-pod --image=nginx:1.25 -n payments
```

---
