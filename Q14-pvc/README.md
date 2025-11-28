# Q14 – Restore MariaDB Deployment with Persistent Storage

This exercise simulates a situation where a MariaDB `Deployment` in the `mariadb` namespace was deleted even though its data should be preserved.  A **`PersistentVolume` (PV)** still exists on the cluster and must be reused through a **`PersistentVolumeClaim` (PVC)**.  You will provision the storage and re‑create the deployment so that the data remains intact.

## 📦 Folder structure

The lab directory now separates **prerequisites** from the **solution**.  The `prerequisites` folder contains manifests to prepare the cluster, and the `solution` folder contains the final YAMLs used to solve the problem.

| Path | Purpose |
| --- | --- |
| `prerequisites/namespace.yaml` | Creates the `mariadb` namespace. |
| `prerequisites/pv.yaml` | (Optional) HostPath PV sized at `250Mi`; reclaim policy is `Retain`. |
| `solution/pvc.yaml` | PVC requesting 250 MiB of storage from the retained PV. |
| `solution/deployment.yaml` | MariaDB deployment that mounts the PVC at `/var/lib/mysql`. |
| `solution/service.yaml` | Cluster‑IP service to expose the database. |
| `README.md` | Explanations, commands, and troubleshooting. |

> **Note:** If a PV already exists in your cluster, you can skip applying `prerequisites/pv.yaml`.  The `storageClassName` on the PVC (`manual`) matches the PV’s `storageClassName`.  Because there is only one PV, Kubernetes will bind the PVC to the retained PV automatically.

## ✅ Objectives

1. Create a `PersistentVolumeClaim` named `mariadb` in the `mariadb` namespace.  It should request **250 MiB** of storage and allow **ReadWriteOnce** access.
2. Update the MariaDB deployment so it mounts the claim at `/var/lib/mysql`.
3. Apply the manifests and verify that the new pod is **running** and **stable** while reusing the retained data volume.

## 🧪 Steps to complete

1. **Ensure the namespace exists.**

   ```bash
   kubectl apply -f prerequisites/namespace.yaml
   kubectl get ns mariadb
   ```

2. **(Optional) Create the PV.**  If your cluster already contains a 250 MiB PV with `ReadWriteOnce` access and the reclaim policy set to `Retain`, you can skip this step.  Otherwise create it:

   ```bash
   # Create a local PersistentVolume for demonstration
   kubectl apply -f prerequisites/pv.yaml

   # View PV status – it should be `Available` until the PVC binds to it
   kubectl get pv mariadb-pv
   ```

3. **Create the PersistentVolumeClaim.**

   ```bash
   kubectl apply -f solution/pvc.yaml

   # Check that the PVC is `Bound` to the PV
   kubectl get pvc mariadb -n mariadb
   kubectl describe pvc mariadb -n mariadb
   ```

4. **Deploy MariaDB using the claim.**

   ```bash
   # Apply the deployment and service
   kubectl apply -f solution/deployment.yaml
   kubectl apply -f solution/service.yaml

   # Watch pod status
   kubectl get pods -n mariadb -w
   ```

   The deployment defines a `volumeMount` named `mariadb-storage` at `/var/lib/mysql` and references the PVC.  When the pod starts it uses the same data directory that was on the old pod.

5. **Verify the solution.**

   ```bash
   # Confirm pod is running and ready
   kubectl get pods -n mariadb

   # Describe the deployment and check events
   kubectl describe deployment mariadb -n mariadb

   # Optionally connect to the MariaDB service
   kubectl get svc mariadb -n mariadb
   ```

6. **Troubleshooting tips.**  If the pod fails to start:

   * Ensure the PVC is bound to the PV and not in `Pending` state.
   * Check that the PV has the `Retain` policy so that deleting the old PVC did not delete the underlying storage.
   * Look at pod events with `kubectl describe pod <pod-name> -n mariadb` for container errors.
   * If the PV does not bind, verify that the PVC’s `storageClassName`, `accessModes`, and size match those of the retained PV.  If necessary, set the PVC’s `volumeName` explicitly to the PV name.

## 📖 `kubectl explain` cheat‑sheet

The table below summarizes key fields from the objects used in this lab.  Use `kubectl explain <resource>` for full documentation.  Only short phrases are provided here to keep the table concise.

| Resource & Path | Type | Description |
| --- | --- | --- |
| `PersistentVolume.spec.capacity` | `map[string]Quantity` | Declared storage capacity (e.g. `250Mi`). |
| `PersistentVolume.spec.accessModes` | `[]string` | Allowed access patterns such as `ReadWriteOnce`. |
| `PersistentVolume.spec.persistentVolumeReclaimPolicy` | `string` | What happens to the volume after the claim is released (`Retain`, `Delete`, etc.). |
| `PersistentVolume.hostPath.path` | `string` | Filesystem path used by a hostPath volume. |
| `PersistentVolumeClaim.spec.accessModes` | `[]string` | Requested access mode from the PV. |
| `PersistentVolumeClaim.spec.resources.requests.storage` | `Quantity` | Amount of storage to allocate (must match PV). |
| `PersistentVolumeClaim.spec.storageClassName` | `string` | Name of the storage class; must equal the PV’s class. |
| `Deployment.spec.replicas` | `int32` | Desired number of pod replicas (1 for this lab). |
| `Deployment.spec.selector.matchLabels` | `map[string]string` | Labels the deployment uses to manage pods. |
| `Deployment.spec.template.spec.containers.image` | `string` | Container image to run (`mariadb:10.9`). |
| `Deployment.spec.template.spec.containers.env` | `[]EnvVar` | Environment variables such as `MYSQL_ROOT_PASSWORD`. |
| `Deployment.spec.template.spec.volumes.persistentVolumeClaim.claimName` | `string` | Name of the PVC bound to this volume. |
| `Service.spec.type` | `string` | Exposure method (`ClusterIP` here). |
| `Service.spec.selector` | `map[string]string` | Determines which pods receive traffic. |
| `Service.spec.ports.port` | `int32` | Exposed port number. |

## 🔬 Verification commands summary

Use the following commands during and after the lab:

```bash
kubectl get pv
kubectl get pvc -n mariadb
kubectl get pods -n mariadb
kubectl describe pv mariadb-pv
kubectl describe pvc mariadb -n mariadb
kubectl describe pod -n mariadb <pod-name>
```

These commands will help you verify that the PVC is bound, the pod is running, and the volume is correctly mounted.  The deployment should be **running** and **stable** after applying the manifests.