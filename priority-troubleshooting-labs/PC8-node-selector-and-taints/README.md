# PC8 – Node Selector and Taints Block Scheduling

Preemption only resolves **resource** shortages.  If a pod cannot be scheduled because it does not match any node’s labels or because of taints/tolerations, preemption never runs【781786337572078†L52-L68】.  You must fix the scheduling constraints instead.  This lab demonstrates both a `nodeSelector` mismatch and a missing toleration.

## Files

| File | Purpose |
|---|---|
| `mismatch-node-selector.yaml` | Pod with a `nodeSelector` that does not exist on any node. |
| `toleration-pod.yaml` | Pod that should run on a tainted node but lacks the corresponding toleration. |
| `priorityclass.yaml` | High‑priority class definition (for completeness). |

## Steps

### A. NodeSelector mismatch

1. **Inspect node labels.**  List node labels using:

   ```bash
   kubectl get nodes --show-labels
   ```

   Confirm that none of your nodes have the label `disktype=ssd`.

2. **Apply the pod with a mismatched `nodeSelector`.**

   ```bash
   kubectl apply -f priorityclass.yaml
   kubectl apply -f mismatch-node-selector.yaml
   ```

   The pod remains in the `Pending` state.  Describe it to see events such as `0/… nodes are available: … didn't match Pod's node selector/affinity`.  Because no node satisfies the selector, the scheduler cannot preempt any pods【781786337572078†L52-L68】.

3. **Resolve the issue.**  Either label a node (`kubectl label node <node> disktype=ssd=true`) or remove the `nodeSelector` from the manifest.  Delete and re‑apply the pod; it should schedule normally.

### B. Taints and tolerations

1. **Taint a node.**  Choose a node and taint it:

   ```bash
   kubectl taint nodes <node> app=blue:NoSchedule
   ```

2. **Deploy a pod without toleration.**

   ```bash
   kubectl apply -f toleration-pod.yaml
   ```

   The pod will remain `Pending` with an event like `node(s) had taint {app: blue}`, and no preemption will occur【781786337572078†L52-L68】.

3. **Add a toleration.**  Edit `toleration-pod.yaml` and add the following under `spec`:

   ```yaml
   tolerations:
   - key: app
     operator: Equal
     value: blue
     effect: NoSchedule
   ```

   Delete the old pod and apply the updated manifest.  The pod should now schedule onto the tainted node.

4. **Remove the taint** (optional) with `kubectl taint nodes <node> app=blue:NoSchedule-`.

## Explanation

When a pod cannot be scheduled due to node selectors, affinity rules, or missing tolerations, the scheduler does not consider preemption.  You must first ensure that the pod can potentially run on at least one node【781786337572078†L52-L68】.  Only then will the scheduler attempt to preempt lower‑priority pods if resources are scarce.
