# PC9 – Unsatisfiable Node/Pod Affinity Prevents Preemption

Preemption can only occur on nodes where a pending pod could run **after** evicting lower‑priority pods.  If a high‑priority pod has **node affinity or inter‑pod affinity/anti‑affinity** rules that cannot be satisfied, the scheduler does not preempt any pods【738076440713976†L1055-L1071】.  This lab creates a pod with unsatisfiable node affinity.

## Files

| File | Purpose |
|---|---|
| `priorityclass.yaml` | Defines a high‑priority PriorityClass. |
| `affinity-pod.yaml` | High‑priority pod with a `requiredDuringSchedulingIgnoredDuringExecution` node affinity that matches no node. |

## Steps

1. **Verify that no node has the required label.**  List node labels:

   ```bash
   kubectl get nodes --show-labels
   ```

   Confirm that none of your nodes have the label `env=production`.

2. **Apply the manifests.**

   ```bash
   kubectl apply -f priorityclass.yaml
   kubectl apply -f affinity-pod.yaml
   ```

   The pod remains in the `Pending` state.  Describe it to see events such as `0/… nodes are available: … node(s) didn't match node selector`.  Because no node meets the affinity rule, preemption is not attempted【738076440713976†L1055-L1071】.

3. **Fix the affinity.**  You can resolve the issue in two ways:

   * **Label a node:**
     ```bash
     kubectl label node <node> env=production=true
     ```
     Once a node has the required label, the pod should schedule (and may preempt lower‑priority pods if there is resource pressure).

   * **Remove or relax the affinity:**  Delete the pod and edit `affinity-pod.yaml` to remove the `affinity` section, then re‑apply.  Without the unsatisfiable constraint, the pod can run normally.

4. **Clean up:**

   ```bash
   kubectl delete -f affinity-pod.yaml -f priorityclass.yaml
   ```

## Explanation

The scheduler only considers nodes where a pod can run if lower‑priority pods are evicted.  An unsatisfied node/pod affinity rule means **no node is a candidate for preemption**, so the scheduler does not evict anything【738076440713976†L1055-L1071】.  Always ensure affinity rules are satisfiable, or provide fallback constraints, before relying on preemption.
