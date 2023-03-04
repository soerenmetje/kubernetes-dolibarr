# Dolibarr in Kubernetes
Run Dolibarr in a Kubernetes cluster. 

[tuxgasys' Dolibarr Docker images](https://hub.docker.com/r/tuxgasy/dolibarr) are used.
Therefor Dolibarr will be installed automatically on first start.

## Prerequisites
- Kubernetes cluster is set up and ready
- Your subdomain points to your Kubernetes cluster
- Kubernetes cluster has a `ClusterIssuer` named `lets-encrypt` set up and ready (may need to be adjusted for your cluster setup)
- Kubernetes cluster has a `StorageClass` named `nfs-csi` set up and ready (may need to be adjusted for your cluster setup)

## Setup
1. Clone this repository and `cd` into it
2. Configure your subdomain in `ingress.yaml` (lines are marked by TODO)
3. Configure passwords in `env-mysql-secret.yaml` and `env-dolibarr-secret.yaml` (lines are marked by TODO)
4. May adjust `ClusterIssuer` in `ingress.yaml` or `StorageClass` in `init/*-persistentvolumeclaim.yaml`
5. Create Kubernetes namespace for dolibarr:
``` bash
kubectl create namespace dolibarr
```
6. Apply configuration to Kubernetes cluster: 
``` bash
kubectl apply -n dolibarr ./init
kubectl apply -n dolibarr .
```
7. Make a tee and wait until auto-install of Dolibarr container is finished (can take 5 to 10 min)
8. Check if Dolibarr webserver is reachable

## Debugging
In case of webserver is not reachable or errors, check status and logs: 
``` bash 
kubectl get all -n dolibarr

kubectl logs -n dolibarr pod/<pod-name>
```