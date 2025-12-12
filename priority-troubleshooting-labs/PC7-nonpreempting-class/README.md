# PC7 – Non‑Preempting `preemptionPolicy: Never`

A PriorityClass can specify `preemptionPolicy: Never`.  Pods using such a class have high priority in the scheduling queue but **never evict other pods**.  They wait until enough resources are freed naturally【738076440713976†L934-L947】.

This lab contrasts a non‑preempting PriorityClass with a regular preempting class.

## Files

| File | Purpose |
|---|---|
| `nonpreempting-class.yaml` | Defines `high-priority-nonpreempting` with `preemptionPolicy: Never`. |
| `nonpreempting-pod.yaml` | High‑priority pod using the non‑preempting class. |
| `low-priority-pods.yaml` | Deployment of low‑priority pods requesting CPU. |
| `priorityclass.yaml` | Defines a regular preempting class (`high-priority-lab`) for comparison. |

## Steps

1. **Apply PriorityClasses and create low‑priority pods.**

   ```bash
   kubectl apply -f priorityclass.yaml
   kubectl apply -f nonpreempting-class.yaml
   kubectl apply -f low-priority-pods.yaml
   ```

   Ensure the low‑priority pods are running and consuming most of the node’s CPU.

2. **Deploy the non‑preempting pod.**

   ```bash
   kubectl apply -f nonpreempting-pod.yaml
   ```

   The pod will remain in the `Pending` state because it does not preempt existing pods.  Use `kubectl describe pod` to verify that no preemption events occur.

3. **Free resources to allow scheduling.**  Delete one or more low‑priority pods or reduce their CPU requests.  After sufficient resources are available, the non‑preempting pod should schedule automatically.

4. **Comparison:**  Delete the non‑preempting pod and deploy a pod using the regular high‑priority class:

   ```bash
   kubectl delete pod nonpreempting-pod
   kubectl run preempting-pod --image=busybox:1.36 --restart=Never --dry-run=client -o yaml \
     --overrides='{"spec":{"priorityClassName":"high-priority-lab","containers":[{"name":"preempting","image":"busybox:1.36","command":["sh","-c","sleep 3600"],"resources":{"requests":{"cpu":"300m","memory":"64Mi"}}}]}}' | kubectl apply -f -
   ```

   The new pod should preempt one or more low‑priority pods and become `Running`.

5. **Clean up:**

   ```bash
   kubectl delete -f low-priority-pods.yaml -f nonpreempting-pod.yaml -f nonpreempting-class.yaml -f priorityclass.yaml
   kubectl delete pod preempting-pod
   ```

## Explanation

When `preemptionPolicy` is set to `Never`, pods using that PriorityClass stay at the front of the scheduling queue but never evict lower‑priority pods【738076440713976†L934-L947】.  This can be useful when you want a job to wait patiently rather than disrupt running workloads.
