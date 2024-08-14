<h1 align="center">
  <img src="https://raw.githubusercontent.com/paradedb/paradedb/dev/docs/logo/readme.svg" alt="ParadeDB" width="368px"></a>
<br>
</h1>

<p align="center">
    <b>Postgres for Search and Analytics</b> <br />
</p>

<h3 align="center">
  <a href="https://paradedb.com">Website</a> &bull;
  <a href="https://docs.paradedb.com">Docs</a> &bull;
  <a href="https://join.slack.com/t/paradedbcommunity/shared_invite/zt-2lkzdsetw-OiIgbyFeiibd1DG~6wFgTQ">Community</a> &bull;
  <a href="https://blog.paradedb.com">Blog</a> &bull;
  <a href="https://docs.paradedb.com/changelog/">Changelog</a>
</h3>

---

[![Publish Helm Chart](https://github.com/paradedb/helm-charts/actions/workflows/publish-helm-chart.yml/badge.svg)](https://github.com/paradedb/helm-charts/actions/workflows/publish-helm-chart.yml)
[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/paradedb)](https://artifacthub.io/packages/search?repo=paradedb)
[![Docker Pulls](https://img.shields.io/docker/pulls/paradedb/paradedb)](https://hub.docker.com/r/paradedb/paradedb)
[![License](https://img.shields.io/github/license/paradedb/paradedb?color=blue)](https://github.com/paradedb/paradedb?tab=AGPL-3.0-1-ov-file#readme)
[![Slack URL](https://img.shields.io/badge/Join%20Slack-purple?logo=slack&link=https%3A%2F%2Fjoin.slack.com%2Ft%2Fparadedbcommunity%2Fshared_invite%2Fzt-2lkzdsetw-OiIgbyFeiibd1DG~6wFgTQ)](https://join.slack.com/t/paradedbcommunity/shared_invite/zt-2lkzdsetw-OiIgbyFeiibd1DG~6wFgTQ)
[![X URL](https://img.shields.io/twitter/url?url=https%3A%2F%2Ftwitter.com%2Fparadedb&label=Follow%20%40paradedb)](https://x.com/paradedb)

# ParadeDB Helm Chart

This repository contains the Helm chart for deploying and managing [ParadeDB](https://github.com/paradedb/paradedb) on Kubernetes via [CloudNativePG](https://cloudnative-pg.io/).

Kubernetes, and specifically the CloudNativePG operator, is the recommended approach for deploying ParadeDB in production. ParadeDB also provides a [Docker image](https://hub.docker.com/r/paradedb/paradedb) and [prebuilt binaries](https://github.com/paradedb/paradedb/releases) for Debian, Ubuntu and Red Hat Enterprise Linux.

## Installing Dependencies

The following dependencies are needed for both production and development.

First, install Helm. See the [Helm docs](https://helm.sh/docs/intro/install/) for more information.

```bash
# macOS
brew install helm

# Ubuntu
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update -y
sudo apt-get install helm -y
```

Then, install the CloudNativePG operator. This chart does not include the Custom Resource Definitions (CRDs) from the CloudNativePG Operator, and it doesn't explicitly depend on it due to Helm's constraints with CRD management. As such, the operator itself is not bundled within this chart and needs to be downloaded manually.

```bash
helm repo add cnpg https://cloudnative-pg.github.io/charts
```

It is also possible to install it directly using the manifest. See the CloudNativePG operator's [installation documentation](https://cloudnative-pg.io/documentation/1.21/installation_upgrade/#installation-on-kubernetes) for more information.

## Usage

The steps below assume you have an accessible Kubernetes cluster running Kubernetes v1.21+.

### Installing the ParadeDB Helm Chart

The recommended installation for the ParadeDB Helm chart is via [Artifact Hub](https://artifacthub.io/packages/helm/paradedb/paradedb). If you prefer to install it manually, first install CloudNativePG onto your Kubernetes cluster.

```bash
helm upgrade --install cnpg --namespace cnpg-system --create-namespace cnpg/cloudnative-pg
```

Then, add the ParadeDB repository to Helm. If you had previously added the repository, instead run `helm repo update` to retrieve the latest versions of the packages. You can then run `helm search repo paradedb` to see the chart and its version.

```bash
helm repo add paradedb https://paradedb.github.io/helm-charts
```

Lastly, install the `paradedb` chart. You can update `<mydatabase>` to the value of your choice.

```bash
helm install <mydatabase> paradedb/paradedb
```

That's it! If you need, you can uninstall the chart with `helm delete <mydatabase>`.

### Connecting to the ParadeDB CloudNativePG K8s Cluster

To connect to your ParadeDB Kubernetes cluster, follow the instructions below. These instructions are also available in `charts/paradedb/templates/NOTES.txt` and are displayed after running the `helm install` command.

First, retrieve the base64-encoded credentials to the Postgres cluster.

```bash
kubectl -n default get secrets paradedb-app -o yaml
```

You can then decode them via `echo '<value>' | base64 --decode`. You need to decode `dbname`, `host`, `password`, `port` and `user`. Then, access your Kubernetes cluster as you normally would and connect via psql, providing the `password` when promoted.

```bash
psql -h <host> -p <port> -U <user> -d <dbname> -W
```

NOTE: If you are trying to access the cluster from outside of Kubernetes, the simplest approach is to forward the Postgres port, `5432`, and connect over `localhost`.

```bash
# Forward the port
kubectl port-forward svc/paradedb-rw 5432:5432

# In a new terminal
psql -h localhost -p 5432 -U <user> -d <mydb> -W
```

### Configuring the ParadeDB Helm Chart

The ParadeDB Helm chart can be configured using the `values.yaml` file or by specifying values on the command line during installation. Check the [values.yaml](https://github.com/paradedb/helm-charts/blob/main/charts/paradedb/values.yaml) file for more information.

## Development

The steps below assume you're looking to develop the ParadeDB Helm chart from scratch.

### Installing Prerequisites

First, install [Minikube](https://minikube.sigs.k8s.io/docs/).

```bash
# macOS
brew install minikube

# Ubuntu
sudo apt-get install minikube -y
```

Then, start your local Kubernetes cluster:

```bash
minikube start
```

Then, install the CloudNativePG CRD:

```bash
helm upgrade --install cnpg --namespace cnpg-system --create-namespace cnpg/cloudnative-pg
```

Then, install the ParadeDB Helm chart:

```bash
git clone https://github.com/paradedb/helm-charts && cd helm-charts/charts/paradedb/
helm install paradedb . --namespace paradedb --create-namespace
```

That's it! You're now ready to start developing the ParadeDB Helm chart. To troubleshoot Kubernetes errors, see [Debugging](#debugging). To access the Postgres cluster, see [Connecting to the ParadeDB CloudNativePG K8s Cluster](#connecting-to-the-paradedb-cloudnativepg-k8s-cluster).

### Debugging

To list the pods, run:

```bash
kubectl get all
```

You will be able to see the status of each pod, which is useful for debugging issues with Postgres' initdb process. To retrieve logs for a specific pod, run:

```bash
kubectl logs <pod>
```

## License

ParadeDB is licensed under the [GNU Affero General Public License v3.0](LICENSE) and as commercial software. For commercial licensing, please contact us at [sales@paradedb.com](mailto:sales@paradedb.com).
