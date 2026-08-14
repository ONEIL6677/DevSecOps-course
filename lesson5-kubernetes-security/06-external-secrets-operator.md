# 6. Git-Safe Secret Management (External Secrets Operator)

## The Real Problem

Kubernetes Secrets are **not safe to store in Git**:

* Base64 encoding is NOT encryption
* Anyone with repo access can decode secrets

Real teams need:

* GitOps-friendly workflows
* Secrets stored **outside** the cluster
* Automatic sync into Kubernetes

This is where **External Secrets Operator (ESO)** fits perfectly.

---

## What is External Secrets Operator (ESO)?

ESO allows Kubernetes to:

* Read secrets from external secret stores
* Inject them as native Kubernetes Secrets

Common backends:

* AWS Secrets Manager
* HashiCorp Vault
* Azure Key Vault

For learning purposes, we will use **Vault** (simple and widely used).

---

## High-Level Flow (Very Important)

1. Secret is stored securely in Vault
2. Git contains only a reference (not the secret)
3. ESO syncs the secret into Kubernetes
4. Pods consume it like a normal Secret

Git stays clean. Secrets stay safe.

---

## Install External Secrets Operator

```
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
```

Verify:

```bash
kubectl get pods -n external-secrets
```

---

## Install Vault

```
kubectl create namespace vault

kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vault
  namespace: vault
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vault
  template:
    metadata:
      labels:
        app: vault
    spec:
      containers:
      - name: vault
        image: hashicorp/vault:1.15
        args: ["server", "-dev"]
        env:
        - name: VAULT_DEV_ROOT_TOKEN_ID
          value: root
        - name: VAULT_DEV_LISTEN_ADDRESS
          value: 0.0.0.0:8200
        ports:
        - containerPort: 8200
EOF
```

Expose Vault:

```bash
kubectl port-forward -n vault deploy/vault 8200:8200
```

---

## Store a Secret in Vault

In another terminal:

```
kubectl exec -it -n vault deploy/vault -- sh

export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=root

vault kv put secret/payments/db username=admin password=SuperSecret123
```

Create Vault Token Secret

```
kubectl create secret generic vault-token \
  --from-literal=token=root \
  -n payments
```

---

## Create a SecretStore (Vault Connection)

```
kubectl apply -f - <<EOF
apiVersion: external-secrets.io/v1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: payments
spec:
  provider:
    vault:
      server: "http://vault.vault.svc.cluster.local:8200"
      path: "secret"
      version: "v2"
      auth:
        tokenSecretRef:
          name: vault-token
          key: token
EOF
```

---

## Create ExternalSecret (Git-Safe)

This is what goes into Git:

```
kubectl apply -f - <<EOF
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: db-secret
  namespace: payments
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: db-secret
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: payments/db
      property: username
  - secretKey: password
    remoteRef:
      key: payments/db
      property: password
EOF
```

Verify:

```bash
kubectl get secrets -n payments
```

---

## Consume the Secret (No Changes for App)

Pods consume it like any Kubernetes Secret:

```
env:
- name: DB_USER
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: username
```

---

## Cleanup

```bash
kind delete cluster --name k8s-security
```
---

