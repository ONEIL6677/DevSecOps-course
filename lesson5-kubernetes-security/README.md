# Kubernetes Security

---

## Prerequisites

* Docker installed
* kubectl installed
* kind installed

Check versions:

```
docker --version
kubectl version --client
kind version
```

---

## Cluster Setup (kind)

Create a dedicated cluster for security demos:

```
kind create cluster --name k8s-security
```

Verify:

```
kubectl get nodes
```