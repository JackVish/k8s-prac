# PC6 – Static / System Pods Cannot Be Preempted

Kubernetes control‑plane components such as `etcd`, `kube‑apiserver`, `kube‑scheduler` and the controller manager run as **static pods** on the master nodes.  These pods use the built‑in `system-node-critical` or `system-cluster-critical` PriorityClasses with extremely high integer values【671162879385945†L306-L313】.  Because they are critical for cluster stability, the scheduler never preempts them.

## Tasks

1. **List PriorityClasses.**

   ```bash
   kubectl get priorityclass
   ```

   You will see built‑in classes such as `system-node-critical` with value `2000001000` and `system-cluster-critical` with value `2000000000`【671162879385945†L306-L313】.  These values are much higher than any user‑defined class.

2. **Inspect static pods.**

   List pods in the `kube-system` namespace:

   ```bash
   kubectl get pods -n kube-system -o wide
   ```

   Then check their priority values:

   ```bash
   kubectl get pod etcd-$(hostname) -n kube-system -o jsonpath='{.spec.priority}'
   ```

   Replace `etcd-…` with the actual pod name from your cluster.  The priority should match the `system-node-critical` or `system-cluster-critical` value.

3. **Attempt to preempt static pods.**

   Create a high‑priority class (e.g. `high-priority-lab` with value `1,000,000`) as in previous labs and deploy a pod that requests large resources.  No matter how high the value is, it will not preempt static pods because their priority is higher and they are critical.

4. **Takeaway**

   Static pods and system add‑ons are protected by high priority classes.  Do not expect preemption to evict them.  Instead, ensure your cluster has sufficient capacity to run both system and user workloads.
