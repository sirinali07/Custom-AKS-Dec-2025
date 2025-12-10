# 🚀 Lab Guide: Deploy HTTPD App to AKS Using Azure DevOps Pipeline

## 🧾 Lab Objective

In this lab, you will:

1. Create a **Kubernetes (K8S) Git repository** to store manifests.  
2. Add the provided **`manifest.yml`** (HTTPD app) to the repo.  
3. Add the provided **pipeline YAML** (e.g. `AKS App Deployment.yml` / `azure-pipelines.yml`) to the repo.  
4. Create a **Kubernetes service connection** in Azure DevOps.  
5. Create and run an **Azure DevOps YAML pipeline** to deploy the app to **AKS**.  
6. Verify the deployment from the **Azure Portal** and via **kubectl**.

---

## ✅ Prerequisites

- An existing **AKS cluster**.
- **kubectl** installed and configured on your machine.
- Access to an **Azure DevOps organization** and **Project**.
- The following files provided to you:
  - `manifest.yml` → Kubernetes deployment + service for HTTPD.
  - `AKS App Deployment.yml` (or similar) → Azure Pipeline YAML file.

> 💡 Trainer Note: Make sure participants know where to download these two files (Teams, LMS, etc.).

---

## 🧱 Step 1: Sign In to Azure DevOps

1. Open a browser and go to:  
   `https://dev.azure.com/`
2. Sign in with your **Azure DevOps** account.
3. Select your **Azure DevOps Organization**.
4. Click on your **Project** where you will work (or create a new Project if needed).

---

## 📁 Step 2: Create a K8S Git Repository for Manifests

You will now create a dedicated **K8S repo** to store Kubernetes manifests and pipeline YAML.

1. In your Azure DevOps project, from the left menu, click **Repos**.
2. At the top, click the **Repo selector** (usually shows “Default” repo name).
3. Click **New repository**.
4. Fill in:
   - **Repository type**: Git
   - **Repository name**: `k8s-aks-httpd` (or any name like `k8s-manifests`)
   - **Add a README**: Optional (Yes/No)
5. Click **Create**.

Your **K8S repo** is now created.

---

## 📂 Step 3: Upload the `manifest.yml` File (HTTPD App)

1. Make sure you are in the **k8s-aks-httpd** repository under **Repos**.
2. Click on **Files** (if not already there).
3. Click **Upload files**.
4. Click **Browse** and select the provided `manifest.yml` file from your local machine.
5. Verify the **file name** is exactly: `manifest.yml`.
6. Add a commit message, e.g.:
   - `Add HTTPD deployment manifest`
7. Click **Commit** (or **Commit to main**).

> 📝 After commit, you should see `manifest.yml` in the root of the repo.

---

## 📄 Step 4: Upload the Pipeline YAML File

You will now add the provided **pipeline definition** file to the same repo.

1. Still in the **k8s-aks-httpd** repository, click **Upload files** again.
2. Click **Browse** and select the pipeline YAML file from your system:
   - Example: `AKS App Deployment.yml`
3. Rename it to **`azure-pipelines.yml`** (recommended for auto-detection), or keep the original name if the trainer instructs so.
4. Add a commit message, for example:
   - `Add AKS HTTPD deployment pipeline`
5. Click **Commit**.

> 🔎 Now the repo should have at least:
> - `manifest.yml`
> - `azure-pipelines.yml` (or `AKS App Deployment.yml`)

---

## 🔐 Step 5: Create Kubernetes Service Connection in Azure DevOps

This service connection allows the pipeline to talk to the AKS cluster.

1. In Azure DevOps, click on the **gear icon** (bottom left) → **Project settings**.
2. Under **Pipelines**, click **Service connections**.
3. Click **New service connection**.
4. Select **Kubernetes** and click **Next**.
5. Choose **Azure Resource Manager** if you want to select AKS from subscription.
6. Authenticate with your Azure subscription if prompted.
7. Select:
   - **Subscription**: Your Azure subscription
   - **Resource group**: One containing your AKS
   - **Kubernetes cluster**: Your AKS cluster
8. Check **Grant access permission to all pipelines**.
9. Give a name such as:  
   - `aks-httpd-connection`
10. Click **Save**.

> 💡 Note this name; it must match the name used in your pipeline YAML.

---

## 🧾 Step 6: Review the Pipeline YAML (High-Level)

> ⚠ Use this section for explanation; the actual YAML file is already in the repo.

A typical pipeline YAML (like your `AKS App Deployment.yml` / `azure-pipelines.yml`) will:

- Use a Microsoft-hosted agent (e.g. `ubuntu-latest`).
- Use `KubernetesManifest@1` task.
- Use the **Kubernetes service connection** created in Step 5.
- Deploy the **`manifest.yml`** file to the AKS cluster.

Example (for explanation only):

```yaml
trigger:
  branches:
    include:
    - main
variables:
- name: kubernetesServiceConnection
  value: 'AKS-Service-Connection'
- name: namespace
  value: 'httpd-app-ns'
- name: manifestsFolder
  value: 'k8s'
stages:
- stage: Deploy
  displayName: Deploy httpd to AKS
  jobs:
  - job: DeployHttpd
    displayName: Deploy httpd app
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: CmdLine@2
      displayName: 'List files'
      inputs:
        script: |
          echo "Listing repo files"
          ls -R
    - task: KubernetesManifest@1
      displayName: 'Deploy httpd manifest to AKS'
      inputs:
        action: 'deploy'
        kubernetesServiceConnection: $(kubernetesServiceConnection)
        namespace: $(namespace)
        manifests: |
          $(manifestsFolder)/manifest.yml

```

## Step 7: Create the Pipeline in Azure DevOps

In Azure DevOps left menu, click Pipelines → Pipelines.

Click New pipeline.

Choose Azure Repos Git.

Select the repo you created:

k8s-aks-httpd

Azure DevOps will ask “Where is your pipeline YAML?”

If your file is named azure-pipelines.yml in the root, it should auto-detect.

Otherwise, select Existing Azure Pipelines YAML file and browse to:

/AKS App Deployment.yml
or

/azure-pipelines.yml

Click Continue or Review to see the YAML.

(Optional) Review variables such as:

kubernetesServiceConnection

namespace

manifests

Click Save and run.

Commit directly to main when prompted and click Save and run again.

🎬 This will start your first pipeline run.

📡 Step 8: Monitor the Pipeline Run

After clicking Save and run, you’ll be taken to the pipeline run details.

Wait for the job DeployHttpd (or similar) to start.

Click on the job to see detailed logs.

Check these key tasks:

Checkout: Should complete successfully.

KubernetesManifest@1 Deploy: Should show Apply successful messages.

Ensure the run finishes with Status: Succeeded.

If there is an error (for example, wrong service connection name or missing manifest.yml), read the error log and fix the YAML or config accordingly, then re-run.

🔍 Step 9: Verify Deployment Using kubectl

🔑 Ensure kubectl is configured to talk to your AKS cluster. If not, use Azure CLI:
az aks get-credentials -g <ResourceGroupName> -n <ClusterName>

Open Terminal / PowerShell on your local machine.

Run to list namespaces and ensure httpd-app exists:



