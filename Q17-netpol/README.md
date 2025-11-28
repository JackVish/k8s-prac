# CKA 2025 – Question 17 Lab: NetworkPolicy for Front‑end ↔ Back‑end Communication

This lab helps you practice creating **Kubernetes NetworkPolicy** objects that allow communication between two Deployments while keeping the overall network as restrictive as possible.  In this scenario we have a `frontend` Deployment and a `backend` Deployment running in separate namespaces.  A default “deny‑all” network policy is already in place for both namespaces, so no pod‑to‑pod traffic is allowed by default.  Your job is to inspect the application components, decide what traffic is required, and create the minimum policies necessary to enable it while leaving all other traffic blocked.

## 🗂 Directory layout

The lab repository is organised into two subfolders:

| Folder | Purpose |
|-------|---------|
| **`prerequisites/`** | YAML files that must be applied before solving the exercise.  These files create the namespaces, Deployments and default `deny‑all` network policies. |
| **`solution/`** | A single YAML manifest that defines the NetworkPolicies necessary for `frontend` and `backend` to talk to each other.  You will apply this after the prerequisites are in place. |

The manifests are numbered to indicate a suggested application order.

## 🚀 Step‑by‑step procedure

### 1. Create namespaces and Deployments

First create the `frontend` and `backend` namespaces and Deployments.  Each Deployment runs an NGINX container on port 80 and is labelled with `app: frontend` or `app: backend` respectively.  Apply all YAML files in the **`prerequisites/`** folder:

```bash
# Apply namespaces, Deployments and deny‑all policies
target_dir=/path/to/q17-lab # update this path accordingly
kubectl apply -f "$target_dir/prerequisites/"

# Check that pods are running and ready
kubectl get pods -n frontend
kubectl get pods -n backend

# Verify that the deny‑all policies exist
kubectl get networkpolicy -n frontend
kubectl get networkpolicy -n backend
```

At this point traffic between the two pods is completely blocked.  You can confirm this by executing a curl from one pod to the other:

```bash
# exec into the frontend pod and attempt to curl the backend Service (will time out due to deny‑all)
pod_front=$(kubectl get pod -n frontend -l app=frontend -o name)
kubectl exec -n frontend "$pod_front" -- curl -m 3 backend.backend.svc.cluster.local
```

### 2. Determine the communication requirements

Open the Deployment manifests in the `prerequisites/` folder and note that both applications expose HTTP on port 80.  No other ports or protocols are used.  We therefore only need to allow TCP port 80 traffic from the frontend pod to the backend pod and vice versa.  According to the Earthly network‑policy guide, an egress policy applied in the frontend namespace can permit traffic to pods in the backend namespace labelled appropriately【293124753925971†L505-L508】.  Likewise, a policy in the backend namespace can allow ingress from the frontend and optional egress back【293124753925971†L555-L558】.

### 3. Create a least‑permissive NetworkPolicy

Apply the file in **`solution/allow-frontend-backend.yaml`**.  This manifest defines **two** NetworkPolicies:

1. **`allow-to-backend`** (namespace `frontend`) – selects pods with `app=frontend` and allows egress only to pods labelled `app=backend` in the `backend` namespace on TCP port 80.
2. **`allow-from-frontend`** (namespace `backend`) – selects pods with `app=backend` and allows both ingress and egress on TCP 80 from/to pods labelled `app=frontend` in the `frontend` namespace.

These policies utilise a combination of `namespaceSelector` and `podSelector` filters so they only match the target pods.  All other traffic remains denied by the existing deny‑all policies.

```bash
kubectl apply -f "$target_dir/solution/allow-frontend-backend.yaml"

# Inspect the policies
kubectl describe networkpolicy allow-to-backend -n frontend
kubectl describe networkpolicy allow-from-frontend -n backend

# Check that communication now works
pod_front=$(kubectl get pod -n frontend -l app=frontend -o name)
pod_back=$(kubectl get pod -n backend -l app=backend -o name)
kubectl exec -n frontend "$pod_front" -- curl -s backend.backend.svc.cluster.local | head -n 2
kubectl exec -n backend "$pod_back" -- curl -s frontend.frontend.svc.cluster.local | head -n 2

# Verify that other traffic is still denied
kubectl exec -n frontend "$pod_front" -- curl -m 3 google.com || echo "External traffic blocked"
```

### 4. Understanding the NetworkPolicy spec (cheat‑sheet)

Use `kubectl explain` to see the fields available for a NetworkPolicy:

```bash
kubectl explain networkpolicy.spec
kubectl explain networkpolicy.spec.ingress
kubectl explain networkpolicy.spec.egress
kubectl explain networkpolicy.spec.podSelector
kubectl explain networkpolicy.spec.policyTypes
```

Key fields include:

| Field | Description |
|------|-------------|
| `podSelector` | Selects the pods in the namespace that the policy applies to.  An empty selector (`{}`) matches all pods. |
| `policyTypes` | Specifies whether the policy applies to `Ingress`, `Egress` or both.  If omitted, it defaults based on the presence of `ingress`/`egress` sections. |
| `ingress` | A list of rules describing what incoming traffic is allowed.  Each rule can specify `from` peers (namespaceSelector/podSelector) and a list of allowed `ports`. |
| `egress` | A list of rules describing what outgoing traffic is allowed.  Each rule can specify `to` peers and `ports` similarly to ingress. |

### 5. Troubleshooting

* **Pods cannot communicate after applying the solution:**
  - Double‑check that the labels in your Deployments match the selectors in your NetworkPolicy (`app: frontend` and `app: backend`).
  - Ensure the deny‑all policies were not accidentally removed; these are necessary so that only traffic specified in your allow rules is permitted.
  - Verify that your CNI plugin supports NetworkPolicy enforcement (most production CNIs such as Calico, Cilium and Weave do).  If using the default Docker bridge network (no CNI), NetworkPolicies will have no effect.
* **Traffic to other services is unexpectedly allowed:**  Only traffic matching the rules in the solution should be allowed.  If you observe other connections going through, verify that no other policies are permitting the traffic and that the deny‑all policies are present.

## ✅ Summary

By applying a pair of targeted NetworkPolicies you allowed the frontend Deployment to talk to the backend Deployment on TCP port 80 while leaving all other ingress and egress blocked.  This approach follows the **least‑privilege** principle.  The Earthly tutorial emphasises that a policy applied in the frontend namespace can allow egress to backend pods and a complementary policy in the backend namespace can allow ingress from the frontend【293124753925971†L505-L508】【293124753925971†L555-L558】.
