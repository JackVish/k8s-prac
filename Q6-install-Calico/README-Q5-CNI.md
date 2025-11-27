# CKA 2025 – Q5: Install a CNI with NetworkPolicy support

**Exam-style task**

> Install and configure a Container Network Interface (CNI) of your choice that meets the specified requirements.  
> Choose one of the following CNI options:
>
> - **Flannel** using the manifest  
>   `https://github.com/flannel-io/flannel/releases/download/v0.26.1/kube-flannel.yml`
> - **Calico** using the manifest  
>   `https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml`
>
> Ensure the selected CNI is properly installed and configured in the Kubernetes cluster.  
> The CNI you choose must:
> - Let Pods communicate with each other
> - **Support NetworkPolicy enforcement**
> - Be installed from manifest files (do not use Helm)

---

## 1. Key exam insight

- **Flannel**: provides basic pod networking but **does _not_ implement Kubernetes NetworkPolicy**.
- **Calico**: provides pod networking **and** NetworkPolicy enforcement.

➡️ **Correct choice for the exam: _Calico_**.

---

## 2. Suggested directory structure for your lab

On the controlplane node (or your lab machine):

```bash
mkdir -p ~/k8s-prac/Q5-CNI-NetworkPolicy/{prereq,solution}
```

The files in this folder correspond to this lab:

- `prereq/01-namespace.yaml` – Namespace for the lab (`cni-lab`)
- `prereq/02-test-deployments.yaml` – Frontend and backend test workloads and a Service
- `solution/01-deny-backend-all.yaml` – Deny all traffic to/from backend pods
- `solution/02-allow-frontend-to-backend.yaml` – Allow only frontend pods to reach backend on port 80

---

## 3. Step-by-step solution (exam-style)

> **Assumption:** This is a fresh cluster created by kubeadm and _no CNI_ is currently installed.  
> If another CNI is installed, it must be removed first in a real exam (not covered here).

### 3.1 Install Calico CNI (using manifests, no Helm)

From the controlplane:

```bash
# 1) Create a folder (optional – for your own organization)
mkdir -p ~/calico
cd ~/calico

# 2) Download the Calico operator manifest (given in question)
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml

# 3) Apply the operator
kubectl apply -f tigera-operator.yaml

# 4) Download Calico custom resources (standard from docs)
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml

# 5) Apply custom resources (this actually configures Calico for this cluster)
kubectl apply -f custom-resources.yaml
```

Wait for all Calico components to be ready:

```bash
kubectl get pods -n calico-system
```

All pods should be `Running` / `Ready`.

Also check that core components have networking:

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
```

Pods should now show pod IPs and `Running`.

---

### 3.2 Create lab namespace and test workloads

Apply the **prerequisite** YAMLs from this folder:

```bash
cd ~/k8s-prac/Q5-CNI-NetworkPolicy

kubectl apply -f prereq/01-namespace.yaml
kubectl apply -f prereq/02-test-deployments.yaml
```

Verify:

```bash
kubectl get pods -n cni-lab -o wide
kubectl get svc -n cni-lab
```

Test that frontend can reach backend:

```bash
FRONTEND_POD=$(kubectl get pod -n cni-lab -l app=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n cni-lab "$FRONTEND_POD" -- wget -qO- http://backend-svc
```

You should see the default NGINX HTML.

---

### 3.3 Verify NetworkPolicy enforcement

Now apply the **solution** NetworkPolicies.

1. **Deny all traffic to/from backend (Ingress + Egress)**

```bash
kubectl apply -f solution/01-deny-backend-all.yaml
```

Re-test from frontend pod:

```bash
FRONTEND_POD=$(kubectl get pod -n cni-lab -l app=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n cni-lab "$FRONTEND_POD" -- wget -qO- http://backend-svc --timeout=3
```

This should now **fail / time out**, showing that NetworkPolicy enforcement works.

2. **Allow only frontend pods to access backend on port 80**

```bash
kubectl apply -f solution/02-allow-frontend-to-backend.yaml
```

Test again:

```bash
FRONTEND_POD=$(kubectl get pod -n cni-lab -l app=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl exec -n cni-lab "$FRONTEND_POD" -- wget -qO- http://backend-svc
```

This should **succeed** now.

Try from another pod (if you create one with a different label) and it should be denied, proving that NetworkPolicy is enforced by the CNI.

---

## 4. What to actually type in the exam

Very compressed, exam-style command sequence (after reading the question):

```bash
# 1) Choose Calico (because it supports NetworkPolicy)
cd /root

curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml
kubectl apply -f tigera-operator.yaml

curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml
kubectl apply -f custom-resources.yaml

# Wait for pods
kubectl get pods -n calico-system

# Optional: create a quick test ns, pods and a NetworkPolicy
# (similar to the YAMLs provided in this lab)
```

If the exam gives you local manifest paths instead of URLs (for offline clusters), just **use those paths instead of curl**.

---

## 5. Files in this lab

- `prereq/01-namespace.yaml` – creates `cni-lab` namespace  
- `prereq/02-test-deployments.yaml` – backend (nginx) + frontend (busybox) + Service  
- `solution/01-deny-backend-all.yaml` – deny all traffic to backend pods  
- `solution/02-allow-frontend-to-backend.yaml` – allow only frontend pods to access backend on port 80

Use these to practice applying and troubleshooting NetworkPolicies after installing the Calico CNI.