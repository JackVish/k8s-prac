# CKA 2025 Q13 – Adjust WordPress Pod Resources

The goal of this lab is to practise adjusting **resource requests** for a multi‑container Deployment so that each Pod receives a fair share of node resources while leaving sufficient headroom for the operating system and cluster services.  You will update a WordPress deployment that runs three replicas in the `relative‑fawn` namespace.

## 🧾 Problem statement

* A WordPress application runs **3 replicas** in the namespace `relative‑fawn`.
* Each Pod currently requests **1 CPU** and **2015360 Ki** of memory (≈1.92 Gi) for every container, including an init container.
* The cluster consists of a **single node** with **4 CPU cores** and **8 Gi** of memory.
* Adjust all Pod resource **requests** so that:
  - Node resources are divided evenly across all three pods.
  - Each pod gets an equal share of CPU and memory.
  - Some overhead is reserved to keep the node stable (this example uses 10%).
  - Use the same requests for both containers and init containers.
* **Do not** modify resource limits (leave them as‑is).

After the update you must confirm that:

* The WordPress Deployment continues to run **3 replicas**.
* All pods are **running** and **ready**.

## 📊 Calculating the new resource requests

### CPU calculation

* The node has **4 CPU cores**, which Kubernetes measures as **4000 m** (millicpu)【538880722553386†L970-L979】.
* Reserve **10 % overhead** for the node itself: 10% of 4000 m = **400 m** reserved.
* Available CPU for pods: 4000 m − 400 m = **3600 m**.
* Divide evenly across 3 pods: 3600 m ÷ 3 = **1200 m** (1.2 CPU) per Pod.

### Memory calculation

* The node has **8 Gi** of memory (8 Gi × 1024 Mi/Gi = **8192 Mi**).
* Reserve **10 % overhead**: 10% of 8192 Mi = **819 Mi** reserved.
* Available memory for pods: 8192 Mi − 819 Mi = **7373 Mi**.
* Divide evenly across 3 pods: 7373 Mi ÷ 3 ≈ **2458 Mi** per Pod.  For simplicity, we round to **2458 Mi**.

### Resulting requests

* **CPU:** `1200m` for each container and init container.
* **Memory:** `2458Mi` for each container and init container.

These values ensure that the three WordPress pods collectively request 3.6 CPU and roughly 7.37 Gi of memory, leaving about 0.4 CPU and 0.8 Gi of memory free on the node for system components.  Kubernetes measures CPU in units where **1 CPU unit equals one core**【538880722553386†L970-L982】 and memory in bytes or with suffixes like `Mi` for mebibytes【538880722553386†L999-L1005】.

## 🛠️ Steps to perform the update

1. **Apply the prerequisite manifests** to create the namespace and the initial deployment:
   ```bash
   kubectl apply -f prereq/01-namespace.yaml
   kubectl apply -f prereq/02-wordpress-deployment.yaml
   ```

2. **Scale down the deployment** to avoid over‑committing resources while editing:
   ```bash
   kubectl scale deployment wordpress --replicas=0 -n relative-fawn
   kubectl wait --for=condition=available --timeout=120s deployment/wordpress -n relative-fawn
   ```

3. **Update the resource requests**.  You can either edit the Deployment in place or apply the provided updated manifest:
   ```bash
   # Option A: apply the updated manifest
   kubectl apply -f solution/01-wordpress-deployment-updated.yaml

   # Option B: patch the existing deployment (equivalent)
   kubectl patch deployment wordpress -n relative-fawn \
     --type='json' -p='[{
       "op": "replace",
       "path": "/spec/template/spec/initContainers/0/resources/requests",
       "value": {"cpu": "1200m", "memory": "2458Mi"}
     },{
       "op": "replace",
       "path": "/spec/template/spec/containers/0/resources/requests",
       "value": {"cpu": "1200m", "memory": "2458Mi"}
     },{
       "op": "replace",
       "path": "/spec/template/spec/containers/1/resources/requests",
       "value": {"cpu": "1200m", "memory": "2458Mi"}
     }]'
   ```

4. **Scale back up** the deployment to 3 replicas:
   ```bash
   kubectl scale deployment wordpress --replicas=3 -n relative-fawn
   kubectl rollout status deployment wordpress -n relative-fawn
   ```

5. **Verify** that all pods are running and ready:
   ```bash
   kubectl get pods -n relative-fawn
   ```
   You should see three pods with the updated requests and `READY` count showing all containers are ready.

## ✔️ Key points to remember

- **Resource requests** help the scheduler decide where to place pods; CPU is measured in millicores (m) and memory in bytes or `Mi`/`Gi`【538880722553386†L970-L980】【538880722553386†L999-L1005】.
- Leave the **resource limits** unchanged unless explicitly asked.
- Scaling down before editing avoids running out of node capacity while updates roll out.
- After the update, verify that the deployment still has three replicas and that all pods are healthy.
