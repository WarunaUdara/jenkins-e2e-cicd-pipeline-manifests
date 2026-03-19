# Kubernetes Manifests for Go Todo Application

This repository contains Kubernetes deployment manifests for the Go Todo REST API application. These manifests are managed by ArgoCD for GitOps-based continuous deployment.

![Kubernetes](https://img.shields.io/badge/Kubernetes-Manifests-326CE5?logo=kubernetes)
![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?logo=argo)
![Auto-Sync](https://img.shields.io/badge/Auto--Sync-Enabled-success)

## Repository Purpose

This is a **manifest-only repository** that serves as the single source of truth for Kubernetes deployments. It follows GitOps principles where:

1. **Jenkins CI/CD** pipeline updates the image tag in `deployment.yaml`
2. **ArgoCD** monitors this repository for changes
3. **Kubernetes** automatically syncs and deploys the updated application

## Files

| File | Description |
|------|-------------|
| `deployment.yaml` | Kubernetes Deployment with 3 replicas, health checks, and resource limits |
| `service.yaml` | LoadBalancer Service for external access on port 80 |
| `README.md` | This documentation file |

## Manifest Details

### Deployment Specification

```yaml
Name: todo-app-go
Replicas: 3
Strategy: RollingUpdate
Image: warunaudara/todo-app-go:<BUILD_NUMBER>
Container Port: 8080
```

#### Features

- **Rolling Updates**: Zero-downtime deployments
- **Health Checks**: 
  - Liveness probe on `/health`
  - Readiness probe on `/ready`
- **Resource Management**:
  - Requests: 64Mi memory, 100m CPU
  - Limits: 128Mi memory, 250m CPU
- **Security**:
  - Non-root user (UID 10001)
  - No privilege escalation
  - Dropped all capabilities

### Service Specification

```yaml
Name: todo-app-go-service
Type: LoadBalancer
External Port: 80
Target Port: 8080
```

## GitOps Workflow

```
Developer Push → Jenkins Pipeline → Docker Build → Image Push
                                          ↓
                              Update deployment.yaml
                                          ↓
                              Git Push to this repo
                                          ↓
                              ArgoCD Detects Change
                                          ↓
                              Kubernetes Sync & Deploy
```

## ArgoCD Configuration

### Application Settings

```yaml
Name: go-todo-app
Project: default
Source:
  repoURL: https://github.com/WarunaUdara/jenkins-e2e-cicd-pipeline-manifests.git
  targetRevision: main
  path: .
Destination:
  server: https://kubernetes.default.svc
  namespace: default
SyncPolicy:
  automated:
    prune: true
    selfHeal: true
```

### Sync Options

- **Auto-Sync**: ✅ Enabled
- **Self-Heal**: ✅ Automatic recovery from manual changes
- **Prune**: ✅ Remove orphaned resources

## Manual Deployment

If you need to deploy manually (without ArgoCD):

```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Apply service
kubectl apply -f service.yaml

# Check deployment status
kubectl get deployments
kubectl get pods -l app=todo-app-go

# Get service external IP
kubectl get service todo-app-go-service

# Test application
EXTERNAL_IP=$(kubectl get service todo-app-go-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl http://$EXTERNAL_IP/health
```

## Updating the Application

### Automated (via Jenkins)

Jenkins automatically updates the image tag on every successful build:

```bash
# Jenkins runs this command
sed -i "s|image: warunaudara/todo-app-go:.*|image: warunaudara/todo-app-go:${BUILD_NUMBER}|g" deployment.yaml
```

### Manual Update

If you need to manually update the image version:

```bash
# Edit deployment.yaml
vim deployment.yaml

# Change the image tag
image: warunaudara/todo-app-go:42  # Change to desired version

# Commit and push
git add deployment.yaml
git commit -m "Update image tag to 42"
git push origin main

# ArgoCD will automatically detect and deploy
```

## Monitoring Deployment

### ArgoCD UI

1. Access ArgoCD dashboard
2. Navigate to `go-todo-app` application
3. View sync status and health
4. Check deployment history

### Kubernetes CLI

```bash
# Watch deployment progress
kubectl rollout status deployment/todo-app-go

# View deployment history
kubectl rollout history deployment/todo-app-go

# Check pod status
kubectl get pods -l app=todo-app-go -w

# View application logs
kubectl logs -l app=todo-app-go --tail=100 -f
```

## Rollback

### Via ArgoCD

```bash
# Rollback to previous version
argocd app rollback go-todo-app

# Rollback to specific revision
argocd app rollback go-todo-app <revision-number>
```

### Via Kubernetes

```bash
# Rollback to previous deployment
kubectl rollout undo deployment/todo-app-go

# Rollback to specific revision
kubectl rollout undo deployment/todo-app-go --to-revision=2
```

## Troubleshooting

### Pods Not Starting

```bash
# Check pod events
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>

# Check deployment events
kubectl describe deployment todo-app-go
```

### Service Not Accessible

```bash
# Verify service
kubectl get service todo-app-go-service

# Check endpoints
kubectl get endpoints todo-app-go-service

# Verify LoadBalancer
kubectl describe service todo-app-go-service
```

### ArgoCD Not Syncing

```bash
# Force sync
argocd app sync go-todo-app

# Refresh app
argocd app get go-todo-app --refresh

# Check sync status
argocd app get go-todo-app
```

## Related Repositories

- **Application Code**: [jenkins-e2e-cicd-pipeline-google-kubernetes-engine](https://github.com/WarunaUdara/jenkins-e2e-cicd-pipeline-google-kubernetes-engine)
- **Docker Image**: [warunaudara/todo-app-go](https://hub.docker.com/r/warunaudara/todo-app-go)

## Contributing

This repository is automatically updated by Jenkins CI/CD pipeline. Manual updates should be made cautiously to avoid sync conflicts.

## License

This project is licensed under the MIT License.

## Author

**Waruna Udara**
- GitHub: [@WarunaUdara](https://github.com/WarunaUdara)

---

**Managed by ArgoCD | Updated automatically by Jenkins**
