# PC4 – Create Resource Pressure to Trigger Preemption

Preemption only happens when a high‑priority pod is **pending** because no node has enough CPU or memory to accommodate it【738076440713976†L969-L1041】.  Without resource pressure, the scheduler simply schedules the pod on a free node.  This lab creates CPU pressure using low‑priority pods and then introduces a high‑priority pod to trigger preemption.

## Files

| File | Purpose |
|---|---|
| `priorityclass.yaml` | Defines two PriorityClasses: a low‑priority class (value `100`) and a high‑priority class (value `1000000`). |
| `low-priority-deployment.yaml` | Deployment that creates multiple low‑priority pods each requesting CPU.  Adjust replicas or CPU requests based on your cluster’s capacity. |
| `high-priority-pod.yaml` | High‑priority pod requesting enough CPU to exceed the remaining capacity of a node. |

## Steps

1. **Create the PriorityClasses.**

   ```bash
   kubectl apply -f priorityclass.yaml
   ```

2. **Deploy the low‑priority pods to fill the node.**

   ```bash
   kubectl apply -f low-priority-deployment.yaml
   ```

   Watch the pods until they are running.  Use `kubectl describe node <node>` and look at the **Allocatable** and **Allocated resources** sections to see how much CPU is free.

3. **Apply the high‑priority pod manifest.**

   ```bash
   kubectl apply -f high-priority-pod.yaml
   ```

   The pod will enter the `Pending` state because there is insufficient CPU.  Describe the pod to see events such as `preempting pods to make room` or `0/1 nodes are available: …`.

4. **Observe preemption.**  Within roughly 20–30 seconds, the scheduler will preempt one or more low‑priority pods to make room for the high‑priority pod.  Use `kubectl get pods` to see that some low‑priority pods are terminated and the high‑priority pod becomes `Running`.  Note that the victims may have a graceful termination period, which introduces a delay【738076440713976†L1032-L1043】.

5. **Clean up** all resources:

   ```bash
   kubectl delete -f high-priority-pod.yaml -f low-priority-deployment.yaml -f priorityclass.yaml
   ```

## Tips

* If your cluster has more than one node, reduce the replica count in `low-priority-deployment.yaml` or add a `nodeSelector` to force all pods onto a single node.
* Adjust the CPU `requests` in the manifests so that the sum of low‑priority pods’ CPU requests nearly saturates a node, leaving just enough room that the high‑priority pod cannot be scheduled without evicting something.
