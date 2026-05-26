# ApplicationSet Demo

Demo GitOps layout for testing Argo CD ApplicationSet from the `main` branch.

```text
applicationset-demo/
├── applicationset.yaml
├── applicationset-platform.yaml
├── main/
│   └── sample-web/
│       ├── deployment.yaml
│       ├── kustomization.yaml
│       ├── namespace.yaml
│       ├── service.yaml
│       └── values.yaml
└── uat/
    └── sample-web/
        ├── deployment.yaml
        ├── kustomization.yaml
        ├── namespace.yaml
        ├── service.yaml
        └── values.yaml
├── platform/
│   ├── ingress-nginx/
│   │   ├── kustomization.yaml
│   │   └── values.yaml
│   └── vault/
│       ├── kustomization.yaml
│       └── values.yaml
```

## Update Repo URL

Edit `applicationset.yaml` and replace:

```text
https://github.com/TranChucThien/gitops-demo.git
```

with your real repository URL.

## Apply

```bash
kubectl apply -n argocd -f argocd/applicationset-demo/applicationset.yaml
```

Apply platform Helm add-ons:

```bash
kubectl apply -n argocd -f argocd/applicationset-demo/applicationset-platform.yaml
```

The ApplicationSet scans the repository root:

```text
main/*
uat/*
```

Each matching folder must contain a `kustomization.yaml`.

The platform ApplicationSet scans the repository root:

```text
platform/*
```

Each platform folder contains a `kustomization.yaml` and `values.yaml`. Kustomize renders the Helm chart with `helmCharts`.

Kustomize Helm rendering requires Argo CD repo-server to run Kustomize with `--enable-helm`. If Argo CD was installed with the values in `argocd/values.yaml`, this is configured by:

```yaml
configs:
  cm:
    kustomize.buildOptions: --enable-helm
```

Apply it with:

```bash
helm upgrade argocd argo/argo-cd \
  --namespace argocd \
  -f argocd/values.yaml
```

The platform ApplicationSet creates Applications:

```text
platform-ingress-nginx -> ingress-nginx namespace
platform-vault         -> vault namespace
```

The Helm values are loaded from each platform folder:

```text
platform/ingress-nginx/values.yaml
platform/vault/values.yaml
```

## Local Ingress Domains

The platform layout includes:

```text
platform/cert-manager/
platform/kyverno/
platform/ingress-automation/
```

`ingress-automation` creates:

```text
ClusterIssuer/local-selfsigned
ClusterPolicy/generate-service-ingress
```

Any Service with this annotation gets an Ingress generated automatically:

```yaml
metadata:
  annotations:
    platform.chucthien.com/ingress: "true"
```

The generated host format is:

```text
<service-name>.<namespace>.ingress.chucthien.com
```

Examples:

```text
sample-web.main-sample-web.ingress.chucthien.com
sample-web.uat-sample-web.ingress.chucthien.com
vault.vault.ingress.chucthien.com
```

The generated Ingress uses `cert-manager.io/cluster-issuer: local-selfsigned`, so cert-manager creates temporary self-signed TLS certificates for local testing.

For local DNS, add each host to `/etc/hosts` on your machine:

```text
<node-ip> sample-web.main-sample-web.ingress.chucthien.com
<node-ip> sample-web.uat-sample-web.ingress.chucthien.com
<node-ip> vault.vault.ingress.chucthien.com
```

Because `/etc/hosts` does not support wildcard domains, use `dnsmasq` or your router/DNS server if you want `*.ingress.chucthien.com` to resolve automatically.
