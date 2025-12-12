# PC2 – Wrong Field Name (`priorityClass` vs. `priorityClassName`)

Kubernetes uses the `priorityClassName` field to look up a pod’s PriorityClass and assign its integer priority【738076440713976†L969-L973】.  Using the wrong field name (`priorityClass`) leaves the pod with the default priority (`0`), so it cannot preempt lower‑priority pods.  This lab helps you spot and fix this mistake.

## Files

| File | Purpose |
|---|---|
| `wrong-priority-field.yaml` | Pod manifest that incorrectly uses `priorityClass` instead of `priorityClassName`. |
| `fixed-priority-field.yaml` | Corrected manifest that uses `priorityClassName`. |
| `priorityclass.yaml` | Defines `high-priority-lab` with a value of `1000000`. |

## Steps

1. **Create the PriorityClass.**  Apply `priorityclass.yaml` to ensure the class exists:

   ```bash
   kubectl apply -f priorityclass.yaml
   ```

2. **Deploy the pod with the wrong field.**

   ```bash
   kubectl apply -f wrong-priority-field.yaml
   ```

   Query the pod’s priority attributes:

   ```bash
   kubectl get pod wrong-field -o jsonpath='{.spec.priorityClassName} {.spec.priority}'
   ```

   You should see an empty `priorityClassName` and a priority of `0`.  Even though a high‑priority class exists, the pod is treated as having the default priority.

3. **Delete and redeploy using the correct field.**

   ```bash
   kubectl delete pod wrong-field
   kubectl apply -f fixed-priority-field.yaml
   ```

   Re‑check the priority values:

   ```bash
   kubectl get pod fixed-field -o jsonpath='{.spec.priorityClassName} {.spec.priority}'
   ```

   The output should be `high-priority-lab 1000000`.

4. **Clean up** the resources after observing the behaviour:

   ```bash
   kubectl delete -f fixed-priority-field.yaml -f priorityclass.yaml
   ```

## Explanation

The admission controller only processes the `priorityClassName` field.  A misspelled or incorrect field results in the pod retaining the default priority (`0`).  You must use `priorityClassName` exactly【738076440713976†L969-L973】.
