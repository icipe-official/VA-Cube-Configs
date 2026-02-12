# VA‑Cube‑Configs

This repository contains the **Helm chart for the Vector Atlas platform** and related configuration templates.  
It is designed to be deployed in a Kubernetes cluster and managed using GitOps tools such as **ArgoCD**.

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
- **Note:** The ingress controller for the platform domain is **not included** in this repository and must be installed separately.

---

## Secrets and Sensitive Data

Sensitive values, such as the **Azure Tenant ID** for workload identity, should **never** be stored in Git.  
Create Kubernetes Secrets in the target cluster and pass them into the chart via ArgoCD Helm parameters or external secret management tools.

---

## Development

Adjust values in `values.yaml` for your cluster and environment.  
Use the templates and app manifests to customize and extend the deployment as needed.  
All deployments should be tracked via ArgoCD GitOps workflows.
