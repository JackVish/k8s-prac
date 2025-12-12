# PC5 – PodDisruptionBudget (PDB) Prevents Preemption

A **PodDisruptionBudget** allows you to limit how many replicas of a deployment can be down at one time.  When preemption tries to evict pods, the scheduler prefers to avoid violating PDBs.  If the PDB’s `minAvailable` is set equal to the number of replicas, the scheduler may fail to find a safe victim to evict【738076440713976†L1045-L1052】.

This lab shows how a strict PDB can block preemption and how relaxing it enables preemption.

## Files

| File | Purpose |
|---|---|
| `priorityclass.yaml` | Defines `high-priority-lab` and `low-priority-lab` PriorityClasses. |
| `low-priority-deployment.yaml` | Creates three low‑priority pods that request CPU. |
| `low-pdb.yaml` | PDB with `minAvailable: 3` for the low‑priority pods.  All three must be running. |
| `high-priority-pod.yaml` | High‑priority pod that needs resources, forcing eviction if possible. |

## Steps

1. **Create the PriorityClasses and low‑priority workload.**

   ```bash
   kubectl apply -f priorityclass.yaml
   kubectl apply -f low-priority-deployment.yaml
   kubectl apply -f low-pdb.yaml
   ```

   Wait until all low‑priority pods are running.  Verify the PDB:

   ```bash
   kubectl get pdb
   kubectl describe pdb low-pdb
   ```

   The PDB should show `min available = 3`, meaning all three pods must be available.

2. **Apply the high‑priority pod.**

   ```bash
   kubectl apply -f high-priority-pod.yaml
   ```

   The pod should remain in the `Pending` state with an event like `0/… nodes are available: … No preemption victims found for incoming pod`.  Since evicting a pod would violate the PDB, the scheduler cannot find any victims【738076440713976†L1045-L1052】.

3. **Relax the PDB.**  Edit `low-pdb.yaml` and set `minAvailable` to `2`, or delete the PDB:

   ```bash
   kubectl delete -f low-pdb.yaml
   # or modify and re‑apply
   sed -e 's/minAvailable: 3/minAvailable: 2/' low-pdb.yaml | kubectl apply -f -
   ```

   Once the PDB allows at least one pod to be evicted, the scheduler should preempt one low‑priority pod to make room for the high‑priority pod.  Observe the high‑priority pod transitioning to `Running`.

4. **Clean up** all resources:

   ```bash
   kubectl delete -f high-priority-pod.yaml -f low-pdb.yaml -f low-priority-deployment.yaml -f priorityclass.yaml
   ```

## Explanation

During preemption, the scheduler attempts to find victims whose **PodDisruptionBudget** would not be violated【738076440713976†L1045-L1052】.  A PDB with `minAvailable` equal to the number of replicas means every pod must remain available, leaving no victim eligible for eviction.  By relaxing `minAvailable` or removing the PDB, the scheduler can preempt lower‑priority pods.
