# Install PX-Backup via ArgoCD (GitOps)

This guide provides step-by-step instructions to install **Portworx PX-Backup** using **ArgoCD** with a GitOps-based approach.

---

## 🧱 Prerequisites

- ArgoCD installed on your Kubernetes cluster
- `argocd` CLI installed on your local machine
- A GitHub repository created with the following directory structure

---

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
│       ├── Charts.yaml
│       ├── templates
│       └── values-px-central.yaml
├── Apps/                       # Your actual applications (e.g., nginx)
│   ├── nginx/
│   │   ├── deployment.yaml
│   │   └── service.yaml
└── argo-apps/                  # ArgoCD Application manifests
    ├── portworx-app.yaml
    └── px-backup-app.yaml
```

---

## 📄 PX-Backup ArgoCD Application (`argo-apps/px-backup-app.yaml`)

This application includes `ignoreDifferences` to prevent false OutOfSync status due to PXBACKUP module being added at runtime.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: px-backup
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/sanjaynaikwadi/gitops-root.git
    path: PX-Backup/PXB
    targetRevision: main
    helm:
      valueFiles:
        - values-px-central.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: px-backup
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
  ignoreDifferences:
    - group: ""
      kind: ConfigMap
      name: pxcentral-frontend-config
      namespace: px-backup
      jsonPointers:
        - /data/FRONTEND_ENABLED_MODULES
    - group: ""
      kind: ConfigMap
      name: pxcentral-ui-configmap
      namespace: px-backup
      jsonPointers:
        - /data/FRONTEND_ENABLED_MODULES
```

---

## 📦 Add Portworx Helm Repository to ArgoCD

```bash
argocd repo add http://charts.portworx.io/ \
  --type helm \
  --name portworx
```

---

## 📅 Clone the PX-Central Helm Chart

Clone the PX-Central chart from the official repo into your GitOps repository:

📌 [https://github.com/portworx/helm/tree/master/charts/px-central](https://github.com/portworx/helm/tree/master/charts/px-central)

**Note:**  
- Don’t copy the `crds/` folder if PXE is already installed or you're on an OpenShift (OCP) cluster.

---

## ⚙️ Generate and Configure `values.yaml`

1. Generate `values.yaml` from [https://central.portworx.com](https://central.portworx.com).
2. Ensure the namespace matches the `spec.destination.namespace` in your ArgoCD application.
3. Modify the following values as needed:

```yaml
isArgoCD: true
pxBackupUIServiceType: "NodePort"   # Optional: if LoadBalancer is not available
pxCentralUIServiceType: "NodePort"  # Optional: if LoadBalancer is not available
```

---

## 🚀 Deploy PX-Backup

Apply your ArgoCD application:

```bash
kubectl apply -f argo-apps/px-backup-app.yaml
```

Then manually sync it via ArgoCD UI or CLI:

```bash
argocd app sync px-backup
```

---

## 🔁 Post-Installation Step (Upgrade Mode)

Once the installation is successful, add the following block to your Helm config to enable upgrade mode:

```yaml
helm:
  valueFiles:
    - values-px-central.yaml
  parameters:
    - name: isUpgrade
      value: 'true'
```

This can be added via the UI or committed to the `px-backup-app.yaml`.

---

## ✅ Done!

You now have PX-Backup deployed and managed using ArgoCD with a GitOps workflow. 🎉
