# ApplicationSet Demo

Demo GitOps layout for testing Argo CD ApplicationSet from the `main` branch.

```text
applicationset-demo/
├── applicationset.yaml
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

The ApplicationSet scans the repository root:

```text
main/*
uat/*
```

Each matching folder must contain a `kustomization.yaml`.
