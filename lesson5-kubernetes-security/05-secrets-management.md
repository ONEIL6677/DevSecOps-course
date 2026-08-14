# 5. Secret Management

## Why Secrets Matter

Never:

* Hardcode passwords
* Commit secrets to Git

Kubernetes Secrets store sensitive data safely (base64 encoded).

---

## Hands-On: Create a Secret

```
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=StrongPassword123 \
  -n dev
```

Verify:

```
kubectl get secrets -n dev
```

---

## Hands-On: Use Secret as Environment Variables

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
  namespace: dev
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
EOF
```

Check:

```
kubectl logs secret-demo -n dev
```

---

## Security Best Practices (Real World)

* Combine Secrets + RBAC
* Restrict secret access per namespace
* Never expose secrets via logs
* Avoid storing plain Kubernetes Secrets in Git

---
