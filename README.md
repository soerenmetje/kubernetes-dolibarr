# Dolibarr in Kubernetes
Run Dolibarr in a Kubernetes cluster. 

[Tuxgasys' Dolibarr Docker image](https://hub.docker.com/r/tuxgasy/dolibarr) is used.
Therefore, Dolibarr will be installed automatically on first start.

## Prerequisites
- Kubernetes cluster is set up and ready
- Your subdomain points to your Kubernetes cluster
- Kubernetes cluster has a `ClusterIssuer` named `lets-encrypt` set up and ready (may need to be adjusted for your cluster setup)

## Setup
1. Clone this repository and `cd` into it
2. Configure your subdomain in `ingress.yaml` (lines are marked by TODO)
3. May adjust `ClusterIssuer` in `ingress.yaml`
4. Configure passwords in `env-mysql-secret.yaml` and `env-dolibarr-secret.yaml` (lines are marked by TODO)
5. May adjust `PersistentVolumeClaim` in `init/*-persistentvolumeclaim.yaml`. Currently, the default storage class of the cluster is used.
6. Create Kubernetes namespace for dolibarr:
``` bash
kubectl create namespace dolibarr
```
7. Apply configuration to Kubernetes cluster: 
``` bash
kubectl apply -n dolibarr -f ./init
kubectl apply -n dolibarr -f .
```
8. Make a tee and wait until auto-install of Dolibarr container is finished (can take 5 to 10 min)
9. Check if Dolibarr webserver is reachable

## Known Issues
- In case special characters are displayed as `?` in generated pdf files: 
  The database should be the collation to `utf8_general_ci`.
  For pdf go to _Home->Setup->Other Setup_ and add this parameter Name-> `MAIN_PDF_FORCE_FONT`  Value-> `dejavusans`

## Troubleshooting
In case of webserver is not reachable or errors, check status and logs: 
``` bash 
kubectl get all -n dolibarr

kubectl logs -n dolibarr pod/<pod-name>
```