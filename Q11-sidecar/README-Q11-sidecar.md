# CKA 2025 Q11 – Adding a Logging Sidecar

This lab shows how to add a *streaming sidecar* to an existing Deployment so that a legacy application can integrate with the Kubernetes logging pipeline.  When applications write logs to files instead of `stdout`, Kubernetes cannot collect them by default.  A **sidecar container** can tail the log file and emit its contents to `stdout/stderr`, making it visible via `kubectl logs`【541902764578219†L92-L97】.  You will patch the `synergy‑deployment` to include a sidecar container that tails a log file and shares it via a volume mount.

## 🧩 Task overview

1. **Create a namespace and sample deployment** – The supplied prerequisite manifests create a `synergy-lab` namespace and a `synergy-deployment` with one container that writes its logs to `/var/log/synergy-deployment.log`.
2. **Add a shared emptyDir volume** – Patch the Deployment to add a `log-volume` and mount it at `/var/log` in both containers.  Do not change the existing container’s image or command; simply add the volume mount.
3. **Add the sidecar container** – Use the `busybox:stable` image and run the command:
   ```bash
   /bin/sh -c "tail -n+1 -f /var/log/synergy-deployment.log"
   ```
   This tails the log file from the shared volume and emits it to `stdout`.  According to the sidecar logging pattern, the sidecar “will tail logs from the associated log file and emit to stdout/stderr”; those logs are then accessible via `kubectl logs`【541902764578219†L92-L97】.

## 📂 Directory structure

```
cka-2025-q11-sidecar/
├── README-Q11-sidecar.md       # this guide
├── prereq/
│   ├── 01-namespace.yaml       # creates the synergy‑lab namespace
│   └── 02-synergy-deployment.yaml # sample deployment writing to a log file
└── solution/
    └── 01-synergy-deployment-patch.json  # JSON patch to add the sidecar and volume
```

## ✅ Steps to reproduce

Follow these steps to simulate the exam environment and verify the solution.

### 1. Apply the prerequisite manifests

Create the namespace and deployment used for this lab:

```bash
kubectl apply -f prereq/01-namespace.yaml
kubectl apply -f prereq/02-synergy-deployment.yaml

# Wait for the pod to start
kubectl get pods -n synergy-lab
```

The deployment creates a single pod running `synergy-app`.  The container writes to `/var/log/synergy-deployment.log` but the file is currently stored in the container’s filesystem and is not exposed to other containers.

### 2. Inspect the current deployment (optional)

You can view the existing pod definition to confirm there is only one container and no volume mounts:

```bash
kubectl get deployment synergy-deployment -n synergy-lab -o yaml
```

### 3. Patch the deployment to add the sidecar and volume

Apply the JSON patch to the deployment.  It will:

- Add an `emptyDir` volume named `log-volume`.
- Mount this volume at `/var/log` for the existing container.
- Add a second container called `sidecar` using the `busybox:stable` image.  It mounts the same volume and tails the log file to `stdout`.

Run the patch command:

```bash
kubectl patch deployment synergy-deployment \
  -n synergy-lab \
  --type=json \
  -p "$(cat solution/01-synergy-deployment-patch.json)"

kubectl rollout status deployment synergy-deployment -n synergy-lab
```

### 4. Verify that logging works

After the rollout completes, you should see two containers in the pod:

```bash
kubectl get pods -n synergy-lab -o wide

# Get logs from the sidecar container
POD=$(kubectl get pods -n synergy-lab -l app=synergy -o jsonpath='{.items[0].metadata.name}')
kubectl logs -n synergy-lab $POD sidecar
```

The output of `kubectl logs` should display the contents of `synergy-deployment.log`.  Because the sidecar tails the file and writes it to `stdout`, Kubernetes collects these logs as part of the pod’s logging stream【541902764578219†L92-L97】.

## 📝 Exam notes

- **Do not modify the main container’s image or command.**  Only add the volume mount to expose `/var/log` and add the sidecar container.
- **Use `busybox:stable`** for the sidecar and run the provided tail command exactly as specified.
- **Use a shared volume** (an `emptyDir` works well) so the log file is accessible to both containers.
- **Do not change other deployments** in the cluster.  Only patch `synergy-deployment`.
- Remember that logs written to `stdout/stderr` are accessible via `kubectl logs`【541902764578219†L92-L97】, which is why the sidecar pattern is effective for file‑based logging.
