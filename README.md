# HabitApp-config

GitOps deployment configuration for [HabitApp](https://github.com/narayanathota/HabitApp).
Argo CD watches this repository and keeps the Kubernetes cluster in sync with it.

```
HabitApp repo --> Jenkins (build, test, scan, image, push)
                     |
                     v  commits new image tag to charts/habitapp/values.yaml
              HabitApp-config repo (this repo)
                     |
                     v  Argo CD detects the change and syncs
                Kubernetes cluster
```

## Layout

| Path | Purpose |
|---|---|
| `charts/habitapp/` | Helm chart: Deployment, Service, ConfigMap, Secret (optional), Ingress (optional), ServiceAccount |
| `argocd/application.yaml` | Argo CD `AppProject` + `Application` (apply once with `kubectl apply -f`) |

## How a deployment happens

1. Jenkins builds `narayanathota/habitapp:<build-number>-<git-sha>` and pushes it to Docker Hub.
2. Jenkins changes only the `image.tag` line in `charts/habitapp/values.yaml` and pushes the commit here.
3. Argo CD sees the new commit and syncs the cluster (rolling update, no downtime).
4. To roll back, `git revert` the tag commit. Argo CD syncs the previous version.

Jenkins never runs `kubectl` or `helm install`.

## Secrets

No secret values live in this repo. The app reads an optional Secret named `habitapp-secret`,
created outside Git:

```bash
kubectl -n habitapp create secret generic habitapp-secret --from-literal=KEY=value
```
