# arcus-portal-config

This repository holds the configuration for the Arcus deployment of the Azimuth portal. The
portal runs on a Kubernetes cluster in a project on the Arcus cloud. As well as hosting
the portal, the Kubernetes cluster is also a Cluster API management cluster that manages itself.

The repository includes two credentials files, which are encrypted using
[git-crypt](https://github.com/AGWA/git-crypt):

  * `clouds.yaml`: application credential for the `rcp-cloud-portal-dev` project on Arcus
  * `kubeconfig`: credentials for the portal Kubernetes cluster

## Cluster configuration

To apply a new cluster configuration, use the `openstack-cluster` chart from
[capi-helm-charts](https://github.com/stackhpc/capi-helm-charts) with the `cluster.yaml`
configuration:

```sh
helm repo add capi https://stackhpc.github.com/capi-helm-chart
helm repo update
# List available versions - stick to tags when possible, or the pre-release versions for main
helm search repo capi --devel --versions
helm upgrade arcus-portal-dev capi/openstack-cluster \
  --namespace capi-self \
  --version <version> \
  --install \
  --values clouds.yaml \
  --set-file cloudCACert=./cert.pem \
  --values cluster.yaml
```

You can follow the progress of the cluster configuration using the
[clusterctl CLI](https://cluster-api.sigs.k8s.io/clusterctl/overview.html):

```sh
clusterctl describe cluster arcus-portal-dev
```

## Portal configuration

The portal is hosted in the same Kubernetes cluster as the Cluster API resources, so the
same `kubeconfig` file is used to deploy the portal.

The portal configuration is applied using the `azimuth` chart from the
[Azimuth](https://github.com/stackhpc/azimuth) project:

```sh
helm repo add azimuth https://stackhpc.github.com/azimuth
helm repo update
# List available versions - stick to tags when possible, or the pre-release versions for main
helm search repo azimuth --devel --versions
helm upgrade azimuth azimuth/azimuth \
  --namespace azimuth \
  --version <version> \
  --install \
  --values portal.yaml
```
