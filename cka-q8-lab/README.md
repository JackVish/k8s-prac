# CKA Lab – Q‑08: Create and Patch PriorityClass

This lab replicates the Certified Kubernetes Administrator (CKA) task described in the question image.  You will create a new `PriorityClass` named `high-priority` with a value one less than the highest existing user‑defined priority class value, then patch an existing deployment to use this new class.  When done correctly, the deployment should roll out with the new priority, causing lower‑priority pods in the same namespace to be evicted.

## Scenario

In the `priority` namespace there is an existing deployment named **busybox‑logger** along with another deployment (**low-service**).  Several user‑defined `PriorityClass` objects already exist: `user-low`, `user-medium` and `critical-user` with values 500, 1000 and 2000 respectively.  Higher integer values correspond to higher scheduling priority【183754255870672†L92-L99】.  System classes like `system-cluster-critical` and `system-node-critical` have very high values and are not relevant here【183754255870672†L111-L123】.

You will create a new priority class with a value **one less** than the highest user-defined value (2000), i.e. 1999.  Then you will patch only the **busybox‑logger** deployment to use this new class and verify that it rolls out successfully.  Do not modify other deployments.

## Files

| File | Purpose |
|---|---|
| `init-priorityclasses.yaml` | Defines the existing user-defined PriorityClasses: `user-low` (500), `user-medium` (1000) and `critical-user` (2000). |
| `priority-namespace.yaml` | Creates the namespace `priority`. |
| `busybox-logger-deployment.yaml` | Deployment with two pods using `user-medium` PriorityClass. |
| `other-deployment.yaml` | Another deployment (`low-service`) using `user-low` PriorityClass. |

## Instructions

1. **Set up the environment.**  Apply the initialization manifests:

   ```bash
   kubectl apply -f init-priorityclasses.yaml
   kubectl apply -f priority-namespace.yaml
   kubectl apply -f busybox-logger-deployment.yaml
   kubectl apply -f other-deployment.yaml
   ```

   Wait until all pods in the `priority` namespace are running:

   ```bash
   kubectl get pods -n priority
   ```

2. **Identify the highest user-defined priority value.**  List the existing PriorityClasses:

   ```bash
   kubectl get priorityclass
   ```

   Note that the highest **user-defined** value is `2000` from `critical-user`.  Higher numbers correspond to higher priority【183754255870672†L92-L99】.

3. **Create a new PriorityClass.**  Create a file `high-priority.yaml` (or use `kubectl create priorityclass`) defining a new class named `high-priority` with `value: 1999`, `globalDefault: false` and an appropriate description.  For example:

   ```yaml
   apiVersion: scheduling.k8s.io/v1
   kind: PriorityClass
   metadata:
     name: high-priority
   value: 1999
   globalDefault: false
   description: "Priority class one point below critical-user"
   ```

   Apply your manifest:

   ```bash
   kubectl apply -f high-priority.yaml
   ```

4. **Patch the busybox‑logger deployment.**  Modify **only** the `busybox-logger` deployment in the `priority` namespace to use the new priority class:

   ```bash
   kubectl -n priority patch deployment busybox-logger \
     --type='json' -p='[{"op":"replace","path":"/spec/template/spec/priorityClassName","value":"high-priority"}]'
   ```

   Alternatively you can run `kubectl edit deployment busybox-logger -n priority` and change `priorityClassName` under `spec.template.spec`.

   After patching, verify that new pods are created with the correct `priorityClassName`:

   ```bash
   kubectl get pod -n priority -l app=busybox-logger -o jsonpath='{.items[*].spec.priorityClassName}'
   ```

5. **Observe preemption.**  Because the new priority value (1999) is higher than `user-low` (500) and `user-medium` (1000) but lower than `critical-user` (2000), the scheduler will evict lower-priority pods from the `low-service` deployment to make room for the updated busybox‑logger pods.  Use:

   ```bash
   kubectl get pods -n priority -w
   ```

   You should see some of the `low-service` pods terminating, followed by new busybox‑logger pods in `Running` state.

6. **Do not modify other deployments.**  Leave the `low-service` deployment definition unchanged; only the busybox‑logger pods should be updated.  Once you have confirmed the rollout and preemption, you can clean up:

   ```bash
   kubectl delete -f busybox-logger-deployment.yaml \
     -f other-deployment.yaml -f init-priorityclasses.yaml -f priority-namespace.yaml -f high-priority.yaml
   ```

## Notes

* The built-in system classes `system-cluster-critical` and `system-node-critical` have extremely high priority values (2,000,000,000+)【183754255870672†L111-L123】 and should not be modified.  User-defined classes should use lower values.
* Higher integer values correspond to higher scheduling priority, and the scheduler orders pending pods accordingly【183754255870672†L92-L99】.
