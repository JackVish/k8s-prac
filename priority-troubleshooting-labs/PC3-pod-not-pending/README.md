# PC3 – Pod Not Pending (CrashLoopBackOff / ImagePullBackOff)

Preemption occurs only when a high‑priority pod is **pending** because there are not enough CPU or memory resources.  If the pod fails for other reasons—such as an invalid image or a runtime crash—the scheduler does not attempt to preempt other pods【781786337572078†L41-L68】.  This lab shows that a pod which is not pending cannot trigger preemption.

## Files

| File | Purpose |
|---|---|
| `priorityclass.yaml` | Defines a high‑priority class named `high-priority-lab` (value `1000000`). |
| `low-priority.yaml` | Deployment of low‑priority pods that fill some CPU resources. |
| `crash-pod.yaml` | High‑priority pod that uses an invalid image tag, causing an `ImagePullBackOff`. |

## Steps

1. **Create the PriorityClass and low‑priority deployment.**

   ```bash
   kubectl apply -f priorityclass.yaml
   kubectl apply -f low-priority.yaml
   ```

   Wait until the low‑priority pods are running.  They consume CPU but do not saturate the node.

2. **Deploy the crashing high‑priority pod.**

   ```bash
   kubectl apply -f crash-pod.yaml
   ```

   Run `kubectl describe pod crash-pod` and observe that the pod enters the `ImagePullBackOff` or `ErrImagePull` state.  It is **not** in the `Pending` phase.

3. **Confirm that preemption does not occur.**  Use `kubectl get pods` to see that none of the low‑priority pods are evicted.  The high‑priority pod is in a crash state, so the scheduler never triggers preemption.

4. **Fix the image and observe scheduling.**

   Edit `crash-pod.yaml` and replace the invalid image tag (`busybox:nonexistent`) with a valid image (`busybox:1.36`).  Delete the crashing pod and re‑apply the corrected file:

   ```bash
   kubectl delete pod crash-pod
   # edit crash-pod.yaml to use a valid image
   kubectl apply -f crash-pod.yaml
   ```

   The high‑priority pod should now be pending (until it can preempt resources) and eventually preempt low‑priority pods once there is resource pressure.

5. **Clean up**:

   ```bash
   kubectl delete -f crash-pod.yaml -f low-priority.yaml -f priorityclass.yaml
   ```

## Explanation

Pods in `ImagePullBackOff` or `CrashLoopBackOff` states are running and therefore no longer considered **pending**.  The scheduler only runs preemption logic when a pod is pending due to insufficient CPU or memory【781786337572078†L41-L68】.  To test preemption, ensure the high‑priority pod remains in the `Pending` phase and that resource pressure exists.
