CKA 2025 – Question 7: Installing Argo CD via Helm

This lab walks through how to install the Argo CD GitOps controller on a
Kubernetes cluster using the official Helm chart. The chart CRDs are
already pre‑installed in the exam environment, so your Helm configuration
must disable CRD installation. As with the other CKA 2025 labs, this
repository contains YAML manifests and a concise exam “cheat sheet” to
practice the task.

📦 What’s included

prereq/01‑namespace.yaml – a manifest that creates the argocd namespace.

solution/argocd‑values.yaml – a Helm values file that disables CRD
installation (installCRDs: false and crds.install: false). Use this
values file both when templating the chart and when performing the
installation.

README‑Q7‑argocd.md – this document, which explains the process and
provides the exam cheat sheet.

✅ Required outcome (from the exam description)

Add the official Argo Helm repository named argo.

Generate a Helm template of the Argo CD chart version 7.7.3 for the
argocd namespace and write it to /argo‑helm.yaml.

Configure the chart so that it does not install CRDs (the exam
environment has already installed them).

Install Argo CD with Helm, release name argocd, version 7.7.3,
namespace argocd and with CRD installation disabled.

You do not need to configure access to the Argo CD server UI for the exam.

🛠️ Step‑by‑step installation

Tip: run all commands from the directory that contains this README.

Create the argocd namespace (or let Helm create it automatically).

kubectl apply -f prereq/01-namespace.yaml
# or, if you prefer to let Helm create it:
# helm install argocd argo/argo-cd --create-namespace --namespace argocd \ 
#   --version 7.7.3 --values solution/argocd-values.yaml


Add the Argo Helm repository and update the local chart index:

helm repo add argo https://argoproj.github.io/argo-helm
helm repo update


Generate the template for the Argo CD chart version 7.7.3 and write
it to /argo-helm.yaml. The following command uses the provided
values file to disable CRD installation. Make sure the / directory is
writeable in your exam environment; if not, choose another path (e.g.
/tmp/argo-helm.yaml).

helm template argocd argo/argo-cd \
  --namespace argocd \
  --version 7.7.3 \
  --values solution/argocd-values.yaml \
  > /argo-helm.yaml

# Quick sanity check: the template should not contain any CRD
# definitions.  You can grep the file for 'CustomResourceDefinition'.
grep -i customresourcedefinition /argo-helm.yaml || echo "✓ no CRDs present"


Install Argo CD via Helm using the same version and values. The
argocd release name is required by the exam. Adding --create-namespace
allows Helm to create the namespace if it does not already exist.

helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  --version 7.7.3 \
  --values solution/argocd-values.yaml


Verify the installation. Wait until the pods are running in the
argocd namespace:

kubectl get pods -n argocd


You should see pods such as argocd-server, argocd-repo-server,
argocd-application-controller and argocd-dex-server in Running or
Pending states. Because access configuration is not required for the
exam, there is no need to port‑forward or expose the argocd-server service.

📝 Exam notes

Use Helm to manage the Argo CD installation. Do not use kubectl apply on the /argo-helm.yaml file; the exam specifically requires a
Helm install.

Ensure the version is set to 7.7.3 and the release name is
argocd. The Helm repository name should be argo.

The CRDs are pre‑installed in the exam environment. Make sure CRD
installation is disabled using the values file (both installCRDs: false
and crds.install: false are provided to handle different chart schema).

The /argo-helm.yaml file generated in step 3 is only needed to satisfy
the exam requirement. You do not need to apply it to the cluster.

Good luck with your studies!