# ArgoCD - Assignment

## Task

- Create a new repo for managing argo files.
- Create an argocd application architecture to deploy k8s manifest files for two clusters. (use application, applicationsets, projects) (at-least 8-10 sample apps should be deployed)
- Deploy a kustomize application to 2 different cluster as prod and stage with one base.
- All the sync should be automated.
- Add git webhook which can trigger the sync as soon as any changes are pushed to repo.
- Should get flock msg for each step of sync Pre, post etc.

---

## Commands

### Installation

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl port-forward svc/argocd-server -n argocd 8080:443
argocd admin initial-password -n argocd
argocd login <ARGOCD_SERVER>
```

### Register a cluster

```bash
kubectl config get-contexts -o name
argocd cluster add docker-desktop
```

### Install ApplicationSet

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/applicationset/v0.4.1/manifests/install.yaml
```

---

## Approach

### For Root Application

├── base
│ └── kustomization.yaml
├── envs
│ ├── production.yaml
│ └── staging.yaml
├── overlays
│ ├── production
│ │ └── kustomization.yaml
│ └── staging
│ └── kustomization.yaml
├── project.yaml
├── README.md
├── root.yaml
|── secret.yaml

- Secret.yaml consists of webhook secret
- project.yaml consists of clusters access
- root.yaml is applied
  - envs - production and staging application are applied
  - It then calls the kustomize files present in overlays folder
  - Which then calls the kustomzation.yaml present in base folder

### For ApplicationSet

├── applicationsetroot.yaml
├── applicationsets
│ ├── prod
│ │ ├── deployments
│ │ │ ├── config.json
│ │ │ ├── mysql-service.yaml
│ │ │ ├── mysql.yaml
│ │ │ ├── redis-service.yaml
│ │ │ ├── redis.yaml
│ │ │ ├── wordpress-service.yaml
│ │ │ └── wordpress.yaml
│ │ └── hooks
│ │ ├── config.json
│ │ ├── failedsync.yaml
│ │ ├── postsync.yaml
│ │ ├── presync.yaml
│ │ └── sync.yaml
│ └── staging
│ ├── deployments
│ │ ├── config.json
│ │ ├── mysql-service.yaml
│ │ ├── mysql.yaml
│ │ ├── redis-service.yaml
│ │ ├── redis.yaml
│ │ ├── wordpress-service.yaml
│ │ └── wordpress.yaml
│ └── hooks
│ ├── config.json
│ ├── failedsync.yaml
│ ├── postsync.yaml
│ ├── presync.yaml
│ └── sync.yaml
├── project.yaml

- ApplicationSet uses git generator approach to create argocd applications
- Values are taken dynamically using go templates

---

## Screenshots

- All screenshots can be found in [screenshots](./screenshots/) folder
