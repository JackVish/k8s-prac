# Kubernetes PriorityClass Troubleshooting Labs

These labs provide hands‑on exercises to practice diagnosing why **PriorityClass‑based preemption** may not work in Kubernetes.  Each lab is isolated in its own folder (`PC1`, `PC2`, …) and contains the manifest files and a `README.md` with step‑by‑step instructions.  You should run these labs in a Kubernetes cluster (for example, a local [`kind`](https://kind.sigs.k8s.io/) or [`minikube`](https://minikube.sigs.k8s.io/docs/) cluster).

## Lab list

| Lab | Scenario | Summary |
|---|---|---|
| **PC1** | Missing PriorityClass | A pod references a PriorityClass that does not exist.  The admission controller rejects the pod until the class is created【738076440713976†L969-L973】. |
| **PC2** | Wrong field name | Demonstrates the difference between `priorityClass` (wrong) and `priorityClassName` (correct).  The wrong field leaves the pod with priority `0`【738076440713976†L969-L973】. |
| **PC3** | Pod not pending | A high‑priority pod crashes due to an invalid image.  Since the pod is not pending, preemption never runs【781786337572078†L41-L68】. |
| **PC4** | Resource pressure | Shows how to create CPU pressure with low‑priority pods and trigger preemption of those pods when a high‑priority pod arrives【738076440713976†L969-L1041】. |
| **PC5** | PodDisruptionBudget | A strict PodDisruptionBudget (PDB) can prevent preemption.  The scheduler tries to honour PDBs but may fail to find victims【738076440713976†L1045-L1052】. |
| **PC6** | Static/system pods | Control‑plane components like `etcd` and `kube‑apiserver` are static pods with extremely high priorities and cannot be preempted【671162879385945†L306-L313】. |
| **PC7** | Non‑preempting PriorityClass | Uses a PriorityClass with `preemptionPolicy: Never`.  Pods in this class wait in the queue and do not evict lower‑priority pods【738076440713976†L934-L947】. |
| **PC8** | Node selectors & taints | If a pod cannot be scheduled because of node selectors, affinity rules or taints, preemption never runs【781786337572078†L52-L68】.  Fix the labels or tolerations instead. |
| **PC9** | Unsatisfiable affinity | A high‑priority pod with unsatisfiable affinity/anti‑affinity rules remains pending.  The scheduler cannot preempt victims because removing them would violate the affinity rules【738076440713976†L1055-L1071】. |
| **PC10** | Preemption timing | Observes the delay caused by graceful termination of victim pods.  Preemption happens, but the high‑priority pod only runs after victims exit【738076440713976†L1032-L1043】. |

Each lab folder contains:

* Kubernetes manifests (`*.yaml`).
* A `README.md` explaining the scenario, commands to run, expected behaviour, and how to resolve the issue.

To get started, change into a lab directory and follow the instructions in its README.  After finishing, clean up resources using `kubectl delete -f <manifest>`.
