# Kubernetes Security

---

## Prerequisites

* Docker installed
* kubectl installed
* kind installed

Check versions:

```bash
docker --version
```
```bash
kubectl version --client
```
```bash
kind version
```

---

## Cluster Setup (kind)

Create a dedicated cluster for security demos:

```bash
kind create cluster --name k8s-security
```

Verify:

```bash
kubectl get nodes
```