# PC1 – Missing `PriorityClass`

This lab demonstrates what happens when a pod references a **PriorityClass** that does not exist.  When the `priorityClassName` in a pod’s specification refers to a class that is missing, the Kubernetes admission controller rejects the pod and it is never created【738076440713976†L969-L973】.

## Files

| File | Purpose |
|---|---|
| `high-priority-pod.yaml` | A simple busybox pod that references a non‑existent PriorityClass named `high-priority-lab`. |
| `priorityclass.yaml` | Defines the missing `PriorityClass` with a high priority value (1,000,000). |

## Steps

1. **Apply the pod manifest first.**

   ```bash
   kubectl apply -f high-priority-pod.yaml
   ```

   You should see an error similar to:

   ```
   error: error validating "high-priority-pod.yaml": error validating data: ValidationError(Pod.spec): unknown field "priorityClass" …
   ```

   or the API server rejects the pod with a message that the `PriorityClass` is not found.  Verify that the pod does **not** appear in the cluster with:

   ```bash
   kubectl get pods
   ```

2. **Check existing PriorityClasses.**

   ```bash
   kubectl get priorityclass
   ```

   You will see only the built‑in classes such as `system-node-critical` and `system-cluster-critical`【671162879385945†L306-L313】.

3. **Create the missing PriorityClass.**

   Apply `priorityclass.yaml` to define `high-priority-lab`:

   ```bash
   kubectl apply -f priorityclass.yaml
   ```

4. **Re-apply the pod manifest.**

   ```bash
   kubectl apply -f high-priority-pod.yaml
   ```

   This time the pod should be created successfully.  Check its priority value with:

   ```bash
   kubectl get pod high-priority-pod -o jsonpath='{.spec.priorityClassName} {.spec.priority}'
   ```

   The output should show `high-priority-lab 1000000`.

5. **Clean up** with:

   ```bash
   kubectl delete -f high-priority-pod.yaml -f priorityclass.yaml
   ```

## Explanation

Kubernetes resolves a pod’s integer priority using the `priorityClassName` field.  If the PriorityClass does not exist, the pod is rejected by the admission controller【738076440713976†L969-L973】.  Creating the appropriate PriorityClass fixes the error.
