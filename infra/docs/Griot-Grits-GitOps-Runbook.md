# Griot & Grits — GitOps Workflow (Argo CD / OpenShift GitOps) + Helm Chart

## Success criteria mapping
- Auto deploy/update after a PR is merged to `main`: enabled via Argo CD `syncPolicy.automated`.
- Drift correction: enabled via `selfHeal: true` and `prune: true`.
- Visibility: Argo CD dashboard shows sync/health for each app.

## Repository layout (requested)
- `gitops/` : GitOps objects (Argo CD Applications)
- `k8s/`    : Helm chart (Kubernetes + OpenShift Routes)
- `argocd/` : AppProject and optional RBAC helper

## Step-by-step (OpenShift)

### 1) Install OpenShift GitOps (if needed)
Verify:
```bash
oc get ns | egrep 'openshift-gitops|openshift-gitops-operator' || true
oc -n openshift-gitops get pods
```

### 2) Grant Argo CD permissions into the target namespace (if required)
```bash
oc apply -f argocd/rolebinding-argocd-to-namespace.yaml
```

### 3) Create the Argo CD project + application(s)
```bash
oc apply -f argocd/appproject-griot-grits.yaml
oc apply -f gitops/apps/griot-grits-dev.yaml
oc apply -f gitops/apps/griot-grits-prod.yaml
```

### 4) Validate status in the GitOps dashboard
```bash
oc -n openshift-gitops get route
```
Open the Argo CD route and confirm **Synced/Healthy**.

### 5) Trigger deployment on merge to `main`
Commit and merge a change to `k8s/helm/griot-grits/values-dev.yaml` (or templates).
Argo CD detects the commit and auto-syncs.

### 6) Prove drift correction
Manually change live state and confirm Argo CD reverts it:
```bash
oc -n gng-user42 scale deploy/griot-grits-backend --replicas=0
```

### 7) Use release tags for reproducibility
In `gitops/apps/griot-grits-prod.yaml`, set:
```yaml
spec:
  source:
    targetRevision: v1.0.0
```

## Helm quickstart (without GitOps)
```bash
helm upgrade --install griot-grits ./k8s/helm/griot-grits -n gng-user42 --create-namespace -f ./k8s/helm/griot-grits/values-dev.yaml
```

## Security note
For real environments, replace plaintext secrets in values with:
- External Secrets Operator / sealed secrets / vault plugin.

## Bibliography (APA)
- Red Hat. (2025). *Installing GitOps*. Red Hat OpenShift GitOps Documentation.
- Argo Project. (n.d.). *Helm*. Argo CD Documentation.
- Argo Project. (n.d.). *Git Generator (ApplicationSet)*. Argo CD Documentation.
- Red Hat. (n.d.). *Universal Base Image 9 Minimal*. Red Hat Ecosystem Catalog.
- Red Hat. (n.d.). *Python 3.11 (UBI9)*. Red Hat Ecosystem Catalog.
- Ahmet Öner. (2025). *Whisper ASR Webservice (Run/Configuration)*. Project documentation.
