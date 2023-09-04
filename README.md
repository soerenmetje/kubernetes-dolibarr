# Dolibarr in Kubernetes
Run Dolibarr in a Kubernetes cluster. 

[Tuxgasys' Dolibarr Docker image](https://hub.docker.com/r/tuxgasy/dolibarr) is used.
Therefore, Dolibarr will be installed automatically on first start.

## Prerequisites
- Kubernetes cluster is set up and ready
- Your subdomain points to your Kubernetes cluster
- Kubernetes cluster has a `ClusterIssuer` named `lets-encrypt` set up and ready (may need to be adjusted for your cluster setup)

## Setup
1. Clone this repository and `cd` into `deploy-kubernetes/`
2. Replace placeholder domain in files with your subdomain:
```bash
DOMAIN_DOLI=bills.test.com
sed -i "s/bills.placeholderdomain.com/${DOMAIN_DOLI}/g" ingress.yaml webserver-deployment.yaml
```
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

### Further Configuration: Whitelist IPs
Especially in production setups security is important. Instead of (or additionally to) Basic Auth, you may want to restrict access by whitelisting certain IPs or IP ranges. This can be configured in the `ingress.yaml` by adding `nginx.ingress.kubernetes.io/whitelist-source-range: "10.0.0.0/16"` to annotations. [This StackOverflow post](https://stackoverflow.com/a/59052066/14355362) provides further details. 

## Advanced Dolibarr Configuration
For further configuration of Dolibarr check out the environment variables in
[tuxgasy/docker-dolibarr Readme](https://github.com/tuxgasy/docker-dolibarr#environment-variables-summary).

## Upgrading Dolibarr Version
1. Backup your volumes - for safety reasons.
2. Remove the `install.lock` file in `/var/www/documents` volume.
3. Ensure that env `DOLI_INSTALL_AUTO` is set to `1` (unset is also fine due to default value is 1). This ensures the migration of the database to the new version.
4. Edit Dolibarr deployment in Kubernetes to use the next newer Dolibarr version.
5. Check Dolibarr webinterface. After Kubernetes completed applying the new configuration, the new version of Dolibarr should be available.

You can still use the standard way to upgrade through web interface.

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

## Bonus: Additional Resources

- Dolibarr Variable substitution system: https://wiki.dolibarr.org/index.php/Variable_substitution_system

## Sources
- https://github.com/tuxgasy/docker-dolibarr