# VA‑Cube‑Configs

This repository contains the **Helm chart for the Vector Atlas platform** and related configuration templates.  
It is designed to be deployed in a Kubernetes cluster and managed using GitOps style with **ArgoCD**.

---

## Repository Structure

- **.github/** – CI/CD workflows for linting, testing, and deploying the chart.  
- **app/** – ArgoCD application manifests (e.g., `vectoratlas.yaml`) that point to this chart.  
- **templates/** – shared Helm template fragments for generating Kubernetes resources.  
- **Chart.yaml** – metadata defining the Helm chart.  
- **values.yaml** – default configuration values for the Helm chart.  
- **.gitignore** – files and directories ignored by Git.  

> The `app/` directory contains ArgoCD application manifests that are tracked via Git for GitOps deployments.

---

## Usage

The Helm chart packages all Kubernetes resources needed to deploy the Vector Atlas platform, including APIs, UI, database components, ingress rules, ConfigMaps, and Secrets.  

Configuration values can be customized in `values.yaml` to adapt the chart to your environment.

---

## Using with ArgoCD (GitOps)

- The repository can serve as the **source for ArgoCD**.  
- First-time deployment should use the **App of Apps pattern** by applying `bootstrap-app.yaml`.  
- Once bootstrapped, ArgoCD will track the application manifests (e.g., `app/vectoratlas.yaml`) directly from Git.  
- All updates to the chart or app manifest should be done via Git; ArgoCD will automatically sync changes.  
- **Note:** The gateway for the platform domain is **not included** in this repository and in infra gitops repo.

---

## Secrets and Sensitive Data

Sensitive values, such as the **Azure Tenant ID** for workload identity, should **never** be stored in Git.  
Create Kubernetes Secrets in the target cluster and pass them into the chart via ArgoCD Helm parameters or external secret management tools.

---

## Development

Adjust values in `values.yaml` for your cluster and environment.  
Use the templates and app manifests to customize and extend the deployment as needed.  
All deployments should be tracked via ArgoCD GitOps workflows.

---

## Multi-Environment Setup (Test & Production)

### Structure

This repository supports separate CI/CD pipelines for **test** and **production** environments, each with its own K3s cluster and ArgoCD instance.

```
app/                         # Test environment
├── helm-values.yaml        # Test-specific values
└── vectoratlas.yaml        # ArgoCD Application for test

app-prod/                   # Production environment  
├── values-prod.yaml        # Production-specific values
└── vectoratlas.yaml        # ArgoCD Application for production
```

### Environment Separation

- **Test**: Uses `app/helm-values.yaml` with Traefik gateway and local `hostPath` PVs
- **Production**: Uses `app-prod/values-prod.yaml` with Envoy gateway and dynamic PVC provisioning
- Both environments reference the **same Helm chart** but with different values
- Changes to test values **do not affect** production and vice versa

### Gateway Provider Support

The chart automatically adapts to the configured gateway provider:
- **Traefik**: Creates Middleware resource and uses ExtensionRef in HTTPRoute
- **Envoy**: Uses standard `urlRewrite.hostname` in HTTPRoute (no Middleware needed)

Configure via: `ingress.gateway.provider` in each environment's values file.

### Conditional Resource Creation

All PVs and PVCs have individual enable flags under `storage.pvs.<name>.enabled` and `storage.pvcs.<name>.enabled`:

- **Test**: `storage.pvs.enabled: true` (local PVs with hostPath)
- **Production**: `storage.pvs.enabled: false` (uses cloud storage classes via PVCs only)

Individual resources (api, public, data, occurencedata) can be enabled/disabled independently.

---

## Validation & Testing

### Dry Run

**Test environment:**
```bash
helm template . -f app/helm-values.yaml
```

**Production environment:**
```bash
helm template . -f app-prod/values-prod.yaml
```

### Deploy to ArgoCD

**Test:**
```bash
kubectl apply -f app/vectoratlas.yaml
```

**Production:**
```bash
kubectl apply -f app-prod/vectoratlas.yaml
```

ArgoCD will automatically sync and deploy the application using the respective values file.
