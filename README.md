## 📁 Recommended Directory Structure

This structure organizes installation files for **Portworx Enterprise** (PXE), **PX-Backup**, and your application workloads. It also includes a dedicated folder for ArgoCD Application manifests.

```
gitops-root/
├── Portworx/
│   └── PXE/                    # PX-Enterprise install
│       ├── values.yaml
│       └── helm-release.yaml   # Optional
├── PX-Backup/
│   └── PXB/                    # PX-Backup install
│       └── values.yaml
├── Apps/                       # Your actual applications (e.g., nginx)
│   ├── nginx/
│   │   ├── deployment.yaml
│   │   └── service.yaml
└── argo-apps/                  # ArgoCD Application manifests
    ├── portworx-app.yaml
    └── px-backup-app.yaml
```
