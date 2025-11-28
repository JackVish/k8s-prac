# CKA 2025 – Question 10: StorageClass and Default Class

This lab demonstrates how to create a **StorageClass** that uses the
existing `rancher.io/local-path` provisioner, sets the
`volumeBindingMode` to `WaitForFirstConsumer` and configures the new
class as the **default** StorageClass in the cluster.  The tasks mirror
Question 10 from the CKA 2025 exam.

## 📦 Contents

* `prereq/01-namespace.yaml` – creates a `storageclass-lab`
  namespace used for testing.
* `prereq/02-test-pvc.yaml` – defines a simple
  `PersistentVolumeClaim` that leaves its `storageClassName` unset.  It
  will bind to whichever StorageClass is marked as the default.
* `solution/01-storageclass-low-latency.yaml` – a manifest defining
  the `low-latency` StorageClass with `WaitForFirstConsumer` mode and
  annotated as the default class【781908377525857†L899-L919】.
* `README-Q10-storageclass.md` – this document with step‑by‑step
  instructions and exam tips.

## ✅ Objective

1. Create a new StorageClass named **low‑latency** using the
   `rancher.io/local-path` provisioner.
2. Set `volumeBindingMode: WaitForFirstConsumer` on the class.  This
   mode defers volume binding until a Pod uses the PVC, allowing the
   scheduler to consider node topology【204471014740212†L1020-L1024】.
3. Make `low‑latency` the **default** StorageClass for the cluster.  Only
   the new class should carry the default annotation; remove the
   annotation from any previously default class【781908377525857†L899-L919】.
4. **Do not modify** existing Deployments or PersistentVolumeClaims.

## 🛠️ Steps

### 1. Apply the prerequisites (optional)

To test that your new default storage class works, you can create a
dedicated namespace and PVC.  Apply the manifests in the `prereq`
folder:

```bash
kubectl apply -f prereq/01-namespace.yaml
kubectl apply -f prereq/02-test-pvc.yaml
```

Verify that the PVC is initially in a `Pending` state because there
may be no default StorageClass yet.  After you create the new
default, re‑create or delete/recreate this PVC to see it bind to a
volume automatically.

### 2. Inspect existing StorageClasses

List the storage classes to determine which one is currently marked as
default:

```bash
kubectl get storageclass
```

The default class is shown with `(default)` or has the annotation
`storageclass.kubernetes.io/is-default-class="true"`【781908377525857†L899-L919】.

### 3. Remove the default annotation from the old StorageClass

To avoid having multiple defaults, patch the existing default storage
class and set its annotation to `false`【781908377525857†L899-L919】:

```bash
# Replace <old-default> with the name of the current default storage class
kubectl patch storageclass <old-default> \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

You can find the current default by running:

```bash
kubectl get sc -o jsonpath='{range .items[?(@.metadata.annotations["storageclass.kubernetes.io/is-default-class"]=="true")]}{.metadata.name}{"\n"}{end}'
```

### 4. Create the low‑latency StorageClass

Apply the manifest in the `solution` folder to create the new class
with the correct provisioner and binding mode【204471014740212†L1020-L1024】:

```bash
kubectl apply -f solution/01-storageclass-low-latency.yaml
```

This manifest sets the annotation
`storageclass.kubernetes.io/is-default-class` to `"true"`, making it
the default class【781908377525857†L899-L919】.  Because the volume binding
mode is `WaitForFirstConsumer`, volume binding will not occur until a
pod uses the claim【204471014740212†L1020-L1024】.

### 5. Verify the new default

List the storage classes again:

```bash
kubectl get storageclass
```

The `low-latency` class should now be marked as `(default)`.  Any PVC
without a `storageClassName` should be dynamically provisioned using
this class.  To test, delete and re‑apply the `test-pvc` in
`storageclass-lab` and observe that it binds to a PersistentVolume.

## 📝 Notes & references

* `WaitForFirstConsumer` delays the binding and provisioning of a
  PersistentVolume until a pod consumes the PVC.  This ensures that
  the scheduler has all the necessary information about node topology
  and constraints before selecting or provisioning a volume【204471014740212†L1020-L1024】.
* Only one StorageClass should typically be marked as the default in a
  cluster.  If more than one is marked default, the most recently
  created default is used for PVCs without a `storageClassName`【781908377525857†L920-L923】.
* Existing deployments and PVCs should not be modified for this task.
  Creating a new StorageClass and adjusting annotations on existing
  classes is sufficient.

Complete these steps to satisfy the requirements of Question 10 and
to gain practice configuring storage classes and defaulting behavior.