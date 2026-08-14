# 3. Network Policies

## What is a NetworkPolicy?

By default:

* All pods can talk to all pods

NetworkPolicy allows:

* Deny all traffic by default
* Explicitly allow required traffic

> NetworkPolicy is **zero-trust networking** for Kubernetes.

---

## Important Note (kind)

kind supports NetworkPolicy **only if a CNI like Calico is installed**.

We will install Calico.

---

## Install Calico on kind

```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

Wait until ready:

```
kubectl get pods -n kube-system
```

---

## Hands-On Setup: Backend and Frontend Pods

We will simulate a real application:

frontend → allowed to access backend

other pods → should be blocked

### Create backend pod:

```
kubectl run backend \
  --image=nginx \
  --labels="app=my-app" \
  -n payments
```

### Create frontend pod:

```
kubectl run frontend \
  --image=busybox \
  --labels="role=frontend" \
  -n payments -- sleep 3600
```

### Create an attacker pod:

```
kubectl run attacker \
  --image=busybox \
  -n payments -- sleep 3600
```

### Expose backend:

```
kubectl expose pod backend \
  --port=80 \
  --name=backend-svc \
  -n payments
```

### Verify Default Behavior (No NetworkPolicy)

Exec into attacker:

```
kubectl exec -it attacker -n payments -- sh
```

Try accessing backend:

`wget -qO- backend-svc`

This works, because Kubernetes allows all traffic by default.

### Apply NetworkPolicy: Allow Specific Traffic Only

```
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific-traffic
  namespace: payments
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
EOF
```

Now we restrict traffic so that:

Only pods with role=frontend can access backend

Only on TCP port 80

### Verify NetworkPolicy Enforcement

From frontend pod:

```
kubectl exec -it frontend -n payments -- sh
wget -qO- backend-svc
```

This should work.

From attacker pod:

```
kubectl exec -it attacker -n payments -- sh
wget -qO- backend-svc
```

This should fail.

---