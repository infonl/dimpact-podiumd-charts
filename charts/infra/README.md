# Infra

## Rationale

Before being able to install any payload helm chart, some infrastructure 
components must be available, such as NFS, Traefik proxy, Cloudnative-PG crds, etc.
You MUST install these first.

## Setup

From the root of this repo, run:

```bash
# (perhaps first:)
# helm repo add cnpg https://cloudnative-pg.github.io/charts
# helm upgrade --install cnpg --namespace cnpg-system --create-namespace cnpg/cloudnative-pg
#
helm dependency build infra charts/infra/
helm template infra charts/infra/ > infra-tpl.yaml
# we must first get the cnpg CRDs installed before the Infra chart can be installed:
yq '. | select(.kind=="CustomResourceDefinition")' < infra-tpl.yaml > infra-crds.yaml
kubectl create -f infra-crds.yaml
# Hand them over to Helm for managent:
for n in backups clusterimagecatalogs clusters databases imagecatalogs \
  poolers publications scheduledbackups subscriptions; do
     crd="crd/${n}.postgresql.cnpg.io"
     kubectl label --overwrite $crd app.kubernetes.io/managed-by=Helm
     kubectl annotate --overwrite $crd \
       meta.helm.sh/release-name=infra meta.helm.sh/release-namespace=default
done

# create payload namespace
kubectl create namespace jenr
# find postgres cluster name and superuser password to prep and apply create-schemas.yaml:
kubectl create -f ~/info/info-podiumd-eu/clouds/gridscale/create-schemas.yaml
```
