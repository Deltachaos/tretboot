# tretboot

A small Kubernetes project to bootstrap your real GitOps in a cluster. I had the challenge to setup and configure a Rancher instance in a empty K3s, without wanting to manually install all the required helm charts in that repository.

I wanted to make the installation as simple as possible, deployed from a Git repository and the only dependency to apply this to a cluster initially should be `kubectl`. This is why I created this small project.

## How to install?

To run `tretboot` you just need apply the `tretboot.yaml` to your cluster and create a config map containing the URL to the Git repository, that contains your Helm charts or `fleet.yaml`.

```
kubectl apply -f https://raw.githubusercontent.com/Deltachaos/tretboot/main/tretboot.yaml
kubectl create configmap tretboot-config \
  --namespace tretboot \
  --from-literal="repository=https://user:password@example.com/somerepo.git"
```

## ConfigMap reference

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tretboot-config
  namespace: tretboot
data:
  repository: "https://user:password@example.com/somerepo.git" # URL to the git repository
  path: "some/path" # Relative path in the git repository to look for your helm charts and fleet bundles
  interval: "60" # Sleep interval before git fetch and upgrade of helm charts
  namespace: "default" # The kubernetes namespace, tretboot installs helmcharts into
  # Custom shell script to be executed before the git clone of the repository (sourced)
  hook-before-clone: |
    echo "Hello World"
  # Custom shell script executed in the reconciliation loop
  hook-loop: |
    echo "Hello World"
```

### Supported resources

Tretboot is looking for directories containing `fleet.yaml` or `Chart.yaml` files. The first matching file in a directory, determines how the directory is treated.

### fleet.yaml

Tretboot implements a minimal feature set of Rancher [fleet.yaml](https://fleet.rancher.io/ref-fleet-yaml) syntax. Currently supported:

```yaml
defaultNamespace: default

helm:
  chart: ./chart # Or a chart name
  repo: https://charts.rancher.io # Only required if chart is not a local chart
  version: 0.1.0 # Only required if chart is not a local chart
  values:
   any-custom: value
```

### Chat.yaml

If tretboot finds a `Chart.yaml` it asumes the directory name as release name for the Helm chart, and installes it into the configured namespace (default namespace is `default`).
