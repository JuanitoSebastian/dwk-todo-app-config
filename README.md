# dwk-todo-app-config

Configuration repository for the **todo-app** project
([DevOps with Kubernetes](https://devopswithkubernetes.com/) course, Exercise 4.10).

Application source code and CI pipelines live in
[JuanitoSebastian/DevOps-With-Kubernetes](https://github.com/JuanitoSebastian/DevOps-With-Kubernetes).
This repository holds the Kubernetes configuration only. ArgoCD syncs the
cluster state from here; the app repo's GitHub Actions workflows build container
images and commit updated image tags into the overlays below.

## Structure

```text
.
├── manifests/
│   ├── base/                  # Kustomize base: deployments, services, Gateway API,
│   │   ├── postgres/          #   NATS ConfigMap, PVC, SOPS-encrypted secrets
│   │   ├── broadcaster-secret.enc.yaml   # Telegram secret (staging + production)
│   │   ├── storage-sa-key.enc.yaml       # GCS backup SA key (production)
│   │   └── todo-db-secret.enc.yaml       # Postgres credentials (staging + production)
│   └── overlays/
│       ├── staging/           # namespace: staging, BROADCASTER_MODE=log-only, no DB backup
│       └── production/        # namespace: production, DB backup CronJob → GCS
└── argocd/
    ├── application-staging.yaml    # ArgoCD Application → overlays/staging → ns staging
    └── application-production.yaml # ArgoCD Application → overlays/production → ns production
```

The SOPS-encrypted secrets are applied **out-of-band** by CI (ArgoCD cannot
decrypt SOPS); they are not referenced from any `kustomization.yaml`.

## Registering with ArgoCD

```bash
kubectl apply -n argocd -f argocd/application-staging.yaml
kubectl apply -n argocd -f argocd/application-production.yaml
```

## Local verification

```bash
kubectl kustomize manifests/overlays/staging
kubectl kustomize manifests/overlays/production
```
