# Question 5 – Install and configure a CNI (Flannel v0.26.1 or Calico v3.28.2)

This folder contains everything needed to answer **Q‑05** of the 2025 CKA project.  
The challenge requires you to install and configure a Container Network Interface (CNI) of your choice and verify that it functions correctly inside a Kubernetes cluster.  
Two CNI plug‑ins are offered:

* **Flannel v0.26.1** (using `kube‑flannel.yml`).  
* **Calico v3.28.2** (using `tigera‑operator.yaml`).

For this solution we chose **Flannel**, because it is lightweight and straightforward to deploy.  The provided manifest matches the official [flannel‑io/flannel release v0.26.1](https://github.com/flannel‑io/flannel/releases/tag/v0.26.1) and uses the `vxlan` backend with the default pod CIDR `10.244.0.0/16`.  If you decide to use Calico instead, see the **Alternative: Calico** section at the bottom for download instructions.

## Files in this directory

| File | Purpose |
| --- | --- |
| `kube‑flannel‑v0.26.1.yaml` | The full Flannel manifest imported from the official `flannel‑io/flannel` repository. Apply this to your cluster to install Flannel. |
| `README.md` | This document explaining how to install and verify the CNI. |

## Prerequisites

* A Kubernetes cluster initialised with `kubeadm` or another installer, **without any existing network plug‑in**.  
  If another CNI is installed (for example, the default `kube‑router` or `calico`), remove it first to avoid conflicts.  For a fresh kubeadm cluster, ensure you **do not** set a pod network via `kubeadm init --pod-network-cidr` because Flannel sets its own network CIDR inside the manifest.

* Administrative access (`kubectl` as a cluster‑admin) to apply cluster‑scoped resources like RBAC roles.

## Installing Flannel

1. Switch to the target context (the exam usually provides context names):

   ```bash
   # example: switch to the cluster named 'k8s'
   kubectl config use-context k8s
   ```

2. Apply the manifest contained in this folder:

   ```bash
   kubectl apply -f kube-flannel-v0.26.1.yaml
   ```

   The file defines the following objects:

   * A dedicated `kube-flannel` namespace.
   * RBAC roles and bindings granting Flannel access to nodes and pods.
   * A `ConfigMap` specifying the CNI and network configuration (default CIDR `10.244.0.0/16`).
   * A `DaemonSet` that runs the flanneld agent on each node.  The init containers install the Flannel CNI plug‑ins into `/opt/cni/bin` and create the network configuration file in `/etc/cni/net.d`.

3. Wait for the pods to become ready:

   ```bash
   kubectl -n kube-flannel get pods -o wide
   ```

   Each node should run a single `kube-flannel-ds` pod in the `Running` state.  Use `kubectl describe pod/<pod-name> -n kube-flannel` if you need to troubleshoot.

4. Verify that pods in your cluster can communicate across nodes.  A simple test is to create a two‑pod `busybox` deployment and ping between them:

   ```bash
   kubectl create deployment busybox --image=busybox --replicas=2 -- /bin/sh -c 'sleep 3600'
   kubectl get pods -o wide
   # note the IP of one busybox pod
   kubectl exec -it <pod-0> -- ping -c 3 <pod-1-ip>
   ```

If the ping succeeds, Flannel is properly installed and the overlay network is functioning.

## Alternative: Calico (optional)

To install Calico instead of Flannel, download the official **v3.28.2** Tigera operator manifest and apply it to your cluster.  Use the raw URL below:

```
https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/tigera-operator.yaml
```

You can save the file through your browser and apply it with:

```bash
kubectl apply -f tigera-operator.yaml
```

The Calico operator will deploy additional CustomResourceDefinitions and components.  After applying, follow the [Calico documentation](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart) to create an `Installation` custom resource specifying the desired pod CIDR and networking backend.  Ensure you remove any existing CNI before deploying Calico.

## Clean‑up (optional)

To uninstall Flannel, delete the resources created from the manifest:

```bash
kubectl delete -f kube-flannel-v0.26.1.yaml
```

This will remove the daemon set and restore the nodes to a state where a different CNI can be installed.