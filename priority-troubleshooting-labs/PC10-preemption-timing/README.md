# PC10 – Observe Preemption Timing and Graceful Termination

Even when preemption is triggered, the high‑priority pod may not start immediately.  When lower‑priority pods are preempted, they receive their **graceful termination period**, which defaults to 30 seconds.  During this time the scheduler waits for the victims to exit【738076440713976†L1032-L1043】.  This lab illustrates the delay and how reducing the termination grace period speeds up preemption.

## Files

| File | Purpose |
|---|---|
| `priorityclass.yaml` | Defines high‑ and low‑priority classes. |
| `low-priority-deployment.yaml` | Deployment of low‑priority pods with `terminationGracePeriodSeconds: 30`. |
| `high-priority-pod.yaml` | High‑priority pod that requires more CPU. |

## Steps

1. **Apply the PriorityClasses and low‑priority deployment.**

   ```bash
   kubectl apply -f priorityclass.yaml
   kubectl apply -f low-priority-deployment.yaml
   ```

   Wait until the low‑priority pods are running.  These pods request CPU and occupy the node.

2. **Apply the high‑priority pod manifest.**

   ```bash
   kubectl apply -f high-priority-pod.yaml
   ```

   The pod enters the `Pending` state.  Run `kubectl describe pod high-priority-timing` and look for events such as `preempting pods to make room`.  Notice that the low‑priority pods receive termination signals.

3. **Watch for delay.**  Run `kubectl get pods` in a separate terminal and watch as the low‑priority pods go into `Terminating` state.  The high‑priority pod will only become `Running` after the victims finish their graceful shutdown (about 30 seconds by default)【738076440713976†L1032-L1043】.

4. **Speed up preemption (optional).**  Edit `low-priority-deployment.yaml` and set `terminationGracePeriodSeconds: 0` or a small value.  Redeploy the low‑priority pods and repeat the test.  You should see that the high‑priority pod schedules almost immediately after eviction.

5. **Clean up**:

   ```bash
   kubectl delete -f high-priority-pod.yaml -f low-priority-deployment.yaml -f priorityclass.yaml
   ```

## Explanation

When the scheduler preempts pods, the victims are not killed instantly.  They are given their `terminationGracePeriodSeconds` to shut down gracefully【738076440713976†L1032-L1043】.  During this grace period the high‑priority pod remains pending.  Reducing the grace period or forcibly terminating pods can speed up preemption, but may cause data loss if the pods need time to exit cleanly.
