# CKA 2025 – Question 9: Expose a Deployment via a NodePort Service

In this lab you will practice modifying a Deployment to expose an
application port and creating a `NodePort` Service to make the pods
accessible externally.  The scenario mirrors Question 9 from the
CKA 2025 exam.

## 📦 Contents

* `prereq/01-namespace.yaml` – creates the `sp-culator` namespace.
* `prereq/02-frontend-deployment.yaml` – defines a Deployment named
  `front-end` with an `nginx` container that does **not** expose port
  80.  You will patch this Deployment to add the port.
* `solution/01-frontend-deployment-patch.json` – a JSON patch that adds
  the `ports` field (port 80/TCP) to the `nginx` container in the
  `front-end` Deployment.
* `solution/02-front-end-svc.yaml` – a Service manifest to expose the
  pods on port 80 and allocate NodePort `30080` within the default
  range (30000–32767)【317790593809807†L1302-L1327】.
* `README-Q9-frontend.md` – this document with step-by-step
  instructions and exam notes.

## ✅ Objective

* Reconfigure the existing Deployment **front-end** in the
  **sp-culator** namespace to expose port **80/TCP** on its `nginx`
  container.
* Create a new Service named **front-end-svc** that targets port 80 on
  the pods and exposes a NodePort so that each pod can be reached
  externally via `<NodeIP>:<NodePort>`【317790593809807†L1302-L1327】.

## 🛠️ Steps

1. **Apply the prerequisites.**  These manifests create the namespace
   and the initial Deployment without a `containerPort`:

   ```bash
   kubectl apply -f prereq/01-namespace.yaml
   kubectl apply -f prereq/02-frontend-deployment.yaml
   
   # Verify that the deployment is running
   kubectl get pods -n sp-culator -l app=front-end
   ```

2. **Patch the Deployment** to expose port 80 on the `nginx` container.
   Use the provided JSON patch to add the `ports` field to the first
   container in the pod template:

   ```bash
   kubectl patch deployment front-end \
     -n sp-culator \
     --type=json \
     --patch-file solution/01-frontend-deployment-patch.json
   
   # Watch the rollout to ensure the change is applied
   kubectl rollout status deployment/front-end -n sp-culator
   ```

3. **Create the NodePort Service** to expose port 80 from the pods and
   allocate a NodePort in the default range.  The manifest sets
   `type: NodePort` and chooses `nodePort: 30080`【317790593809807†L1302-L1327】:

   ```bash
   kubectl apply -f solution/02-front-end-svc.yaml
   
   # Check the assigned NodePort and ClusterIP
   kubectl get svc front-end-svc -n sp-culator
   ```

4. **Test connectivity (optional).**  Retrieve a node IP and the
   allocated NodePort, then curl the service.  This requires access
   to the cluster nodes from your terminal:

   ```bash
   NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
   NODE_PORT=$(kubectl get svc front-end-svc -n sp-culator -o jsonpath='{.spec.ports[0].nodePort}')
   curl http://$NODE_IP:$NODE_PORT
   ```

   You should see the default nginx welcome page.

## 📝 Notes & references

* A NodePort Service exposes the service on each node’s IP at a port
  number chosen from the range `30000–32767`, or a specific value
  provided in the manifest【317790593809807†L1302-L1327】.  Traffic sent to
  `<NodeIP>:nodePort` is forwarded to one of the service’s endpoints.
* To specify a particular NodePort, set the `nodePort` field in the
  Service’s port spec.  The control plane will allocate that port if
  it is available; otherwise the API call will fail【317790593809807†L1323-L1327】.
* Changing a Deployment’s container ports triggers a rollout; make sure
  to monitor the rollout status with `kubectl rollout status` to
  ensure the new pods are running successfully.

Complete these steps and you’ll have reconfigured the deployment and
exposed it via NodePort as required by Question 9.