# Q3 – HorizontalPodAutoscaler Lab (CKA 2025)

This folder contains manifests to practice creating an HPA named `apache-server` in the `autoscale` namespace, targeting the Deployment `apache-server`.

## Requirements
- CPU target: **50%**
- minReplicas: **1**
- maxReplicas: **4**
- Downscale stabilization window: **30 seconds**

## Apply files
```
kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-apache-deployment.yaml
kubectl apply -f 02-apache-service.yaml
kubectl apply -f 03-hpa-apache-server.yaml
```

## Verify
```
kubectl get deploy,svc,hpa -n autoscale
kubectl describe hpa apache-server -n autoscale
```

## R&D – Generate CPU load
```
kubectl run loader -n autoscale --image=busybox:1.36 --restart=Never -- \
  sh -c 'while true; do wget -q -O- http://apache-server.autoscale.svc.cluster.local > /dev/null; done'
```

Then watch scaling:
```
kubectl get hpa -n autoscale -w
```
