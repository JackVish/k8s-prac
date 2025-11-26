# Q4 – Ingress Echo Lab with Ingress Controller (CKA 2025)

This folder extends the Q4 echo service lab by including instructions for
installing an **Ingress controller**.  In Kubernetes, an Ingress resource
defines routing rules, but those rules are not enforced until a controller
observes and implements them.  Without a controller, your `Ingress` objects
will be created in the API but no traffic will be forwarded.

The manifest files here are the same as the original Q4 lab.  They
deploy a simple echo server and expose it via an Ingress.  Use the
additional instructions below to ensure the Ingress actually works.

## Files Included

| File | Purpose |
| --- | --- |
| `00-namespace.yaml` | Creates the namespace `echo-sound`. |
| `01-echoserver-deployment.yaml` | Deploys a single Pod using the `k8s.gcr.io/echoserver:1.4` image listening on port 8080. |
| `02-echoserver-service.yaml` | Defines a ClusterIP Service named `echoserver-service` exposing the deployment on port 8080. |
| `03-echo-ingress.yaml` | Creates an Ingress named `echo` with host `example.org` and path `/echo` routing to the service on port 8080. |

## Installing the NGINX Ingress Controller

If your cluster doesn’t already have an Ingress controller, you need to
install one so that your `Ingress` rules take effect.  The official
Kubernetes documentation recommends deploying the **NGINX Ingress
Controller** by applying its published manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/cloud/deploy.yaml
```

Running this command creates the `ingress-nginx` namespace along with the
roles, service accounts, deployments and services required for the
controller【600294766089745†screenshot】.

### Steps

1. **Install the controller** by applying the recommended manifest:

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/cloud/deploy.yaml
   ```

2. **Deploy the lab workloads**:

   ```bash
   kubectl apply -f 00-namespace.yaml
   kubectl apply -f 01-echoserver-deployment.yaml
   kubectl apply -f 02-echoserver-service.yaml
   kubectl apply -f 03-echo-ingress.yaml
   ```

3. **Wait for the controller to be ready**.  You can check the status of the
   controller Pod with:

   ```bash
   kubectl -n ingress-nginx get pods
   ```

4. **Test the Ingress**.  Map `example.org` to the external IP or
   port of the Ingress controller and then run:

   ```bash
   curl -o /dev/null -s -w "%{http_code}\n" http://example.org/echo
   ```

   The command should return `200` when the echo server is reachable.

### Note

On local clusters like minikube you might need to run `minikube tunnel` or use
`minikube service -n ingress-nginx ingress-nginx-controller` to expose the
Ingress controller.  In other environments, configure DNS or your
`/etc/hosts` file to resolve `example.org` to the controller’s IP address.