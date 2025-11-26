# CKA 2025 Question 01 – NGINX TLS Config

This folder contains lab files for a **ConfigMap/Pod/VolumeMount/Deployment** question to practice for the Certified Kubernetes Administrator (CKA) exam.

## Scenario

An NGINX Deployment named `nginx-static` is running in the `nginx-static` namespace. It is configured using a ConfigMap named `nginx-config`. Your task is to **update the configuration to allow only TLSv1.3 connections**. After editing the ConfigMap, you must recreate or restart the necessary resources so that the change takes effect.

To test your changes use the following command from within the cluster (for example, from a temporary curl pod):

```sh
candidate@cka2025$ curl --tls-max 1.2 https://web.k8s.local
```

The command should **fail** because TLSv1.2 is no longer allowed.

## Contents

The following manifests are provided to build the lab environment:

| File | Purpose |
| --- | --- |
| `00-namespace.yaml` | Creates the `nginx-static` namespace. |
| `01-nginx-configmap.yaml` | Defines the initial `nginx-config` ConfigMap with TLSv1.2 and TLSv1.3 enabled. This is the file you will edit to allow only TLSv1.3. |
| `02-tls-secret.yaml` | Defines a placeholder TLS secret. In practice you can generate a self‑signed certificate with `openssl` and create the secret via `kubectl create secret tls`. |
| `03-nginx-deployment.yaml` | Creates the `nginx-static` Deployment which mounts the ConfigMap and TLS secret. |
| `04-nginx-service.yaml` | Exposes the deployment via a ClusterIP service named `web` on port 443. |

To set up the lab, apply the files in order:

```sh
kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-nginx-configmap.yaml
kubectl apply -f 02-tls-secret.yaml   # or create via kubectl create secret tls ...
kubectl apply -f 03-nginx-deployment.yaml
kubectl apply -f 04-nginx-service.yaml
```

Edit `01-nginx-configmap.yaml` to remove `TLSv1.2` from the `ssl_protocols` directive, then restart the deployment:

```sh
kubectl -n nginx-static edit configmap nginx-config
# change: ssl_protocols TLSv1.2 TLSv1.3;
# to:     ssl_protocols TLSv1.3;
kubectl -n nginx-static rollout restart deploy/nginx-static
```

Finally, verify that TLSv1.2 is rejected and TLSv1.3 succeeds.

NOTE: Below are openssl command to create cert and key and kubectl commands to create a secret
```sh
openssl req -x509 -nodes -days 365 -newkey rsa:2048   -keyout tls.key -out tls.crt   -subj "/CN=web.k8s.local"

kubectl create secret tls nginx-tls --namespace nginx-static --cert=tls.crt --key=tls.key
```