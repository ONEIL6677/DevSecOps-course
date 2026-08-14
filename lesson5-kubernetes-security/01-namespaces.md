# 1. Namespaces

## What is a Namespace?

A namespace is a **logical isolation boundary** inside a Kubernetes cluster.

Namespaces help with:

* Team or project separation (payments, search, platform)
* Team isolation
* Applying security controls (RBAC, NetworkPolicy, quotas)

> Kubernetes security always starts with namespaces.

---

## Hands-On: Creating Namespaces

Create namespaces for teams/projects:

```
kubectl create namespace payments
kubectl create namespace search
```

Verify:

```
kubectl get namespaces
```

---

## Hands-On: Deploying Apps in Different Namespaces

Create a simple nginx deployment in `payments`:

```
kubectl create deployment nginx-payments --image=nginx -n payments
```

Create the same in `search`:

```
kubectl create deployment nginx-search --image=nginx -n search
```

Verify:

```
kubectl get pods -n payments
kubectl get pods -n search
```

**Key takeaway:**

* Namespaces map cleanly to teams or projects
* Isolation helps apply RBAC, NetworkPolicy, and quotas per team

Namespaces do NOT provide security by default (yet)

---