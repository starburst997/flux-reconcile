# FluxCD Reconcile

A GitHub Action that triggers FluxCD reconciliation for HelmChart sources and HelmReleases in your Kubernetes cluster.

## Features

- 🔄 Automatically reconciles HelmChart sources and HelmReleases
- ⚡ Fast and reliable deployment updates
- 🔐 Secure kubectl configuration
- ⏱️ Configurable timeouts for both chart and release reconciliation

## Usage

```yaml
- name: Trigger FluxCD Reconciliation
  uses: starburst997/flux-reconcile@v1
  with:
    kube-config: ${{ secrets.KUBE_CONFIG }}
    chart-name: my-app-my-app
    registry-namespace: flux-registry
    release-name: my-app
    release-namespace: my-app
```

### Complete Example

```yaml
name: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build and push Helm chart
        run: |
          # Your build and push steps here
          helm package .
          helm push *.tgz oci://registry.example.com/charts

      - name: Trigger FluxCD Reconciliation
        uses: starburst997/flux-reconcile@v1
        with:
          kube-config: ${{ secrets.KUBE_CONFIG }}
          chart-name: my-app-my-app
          registry-namespace: flux-registry
          release-name: my-app
          release-namespace: my-app
          chart-timeout: 3m
          release-timeout: 10m
```

## Inputs

| Input                | Description                                             | Required | Default |
| -------------------- | ------------------------------------------------------- | -------- | ------- |
| `kube-config`        | Kubernetes configuration (base64 encoded or plain text) | Yes      | -       |
| `chart-name`         | Name of the HelmChart to reconcile                      | Yes      | -       |
| `registry-namespace` | Namespace where the HelmChart source is located         | Yes      | -       |
| `release-name`       | Name of the HelmRelease to reconcile                    | Yes      | -       |
| `release-namespace`  | Namespace where the HelmRelease is located              | Yes      | -       |
| `chart-timeout`      | Timeout for HelmChart reconciliation                    | No       | `2m`    |
| `release-timeout`    | Timeout for HelmRelease reconciliation                  | No       | `5m`    |

## How It Works

This action performs the following steps:

1. **Setup kubectl** - Installs and configures kubectl with your cluster credentials
2. **Install FluxCD CLI** - Ensures the Flux CLI is available (skips if already installed)
3. **Reconcile HelmChart** - Triggers reconciliation of the HelmChart source to fetch the latest chart from your OCI registry
4. **Reconcile HelmRelease** - Triggers reconciliation of the HelmRelease to deploy the updated chart

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Links

- [GitHub Repository](https://github.com/starburst997/flux-reconcile)
- [GitHub Marketplace](https://github.com/marketplace/actions/fluxcd-reconcile)
- [FluxCD Documentation](https://fluxcd.io/)
