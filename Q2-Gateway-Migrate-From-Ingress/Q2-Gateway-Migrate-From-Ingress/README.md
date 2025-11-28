# Q2 – Gateway API Migration Lab (CKA Edition)

This folder contains Kubernetes manifests to practice migrating an existing HTTPS Ingress into a Gateway and HTTPRoute configuration.  These files mirror the scenario described in the exam-style question: starting with an Ingress named `web` and moving to a Gateway named `web-gateway` with a Route named `web-route`, while maintaining HTTPS and the same routing rules.

## Files Included

| File | Description |
| --- | --- |
| `00-web-deploy-svc.yaml` | Deployment and ClusterIP Service for a simple NGINX pod serving HTTP on port 80. |
| `01-web-tls-secret.yaml` | Placeholder TLS secret definition. Replace the `tls.crt` and `tls.key` fields or generate the secret via `kubectl`. |
| `02-web-ingress.yaml` | Existing Ingress that routes `https://web.k8s.local/` to the `web-svc` service using the `web-tls` secret. |
| `03-gatewayclass-nginx.yaml` | GatewayClass named `nginx`, matching the exam question. |
| `04-web-gateway.yaml` | Gateway configured to listen on HTTPS port 443 for the hostname `gateway.web.k8s.local` and terminate TLS using `web-tls`. |
| `05-web-httproute.yaml` | HTTPRoute that binds to `web-gateway`, routes all paths to the `web-svc` backend, and restricts hostnames to `gateway.web.k8s.local`. |

## Notes – TLS Setup for R&D

### Generate a TLS certificate using OpenSSL

To generate a self‑signed certificate and key for `gateway.web.k8s.local`, run the following command on your workstation or in a test container:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=gateway.web.k8s.local"
```

### Create the TLS secret

You can either populate the `01-web-tls-secret.yaml` file by base64‑encoding the certificate and key, or create the secret directly with kubectl:

```bash
kubectl create secret tls web-tls \
  --cert=tls.crt --key=tls.key \
  --namespace default
```

Applying the manifests in this folder in order will simulate the state before and after migrating from Ingress to Gateway API.  After creating the Gateway and Route, you can delete the Ingress resource named `web` to complete the migration.
