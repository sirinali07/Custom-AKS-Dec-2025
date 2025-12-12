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

## 📂 Step 3: Create the `manifest.yml` File (HTTPD App)

1. Make sure you are in the **k8s-aks-httpd** repository under **Repos**.
2. Click on **Files** .
3. Click **New** → **Folder**.
4. Type *New forlder name* (i.e `K8S` ) and *New filename* (i.e `azure-pipeline.yaml`)
5. Click **create** and  the provided  file content as mentioned below:
   
```yaml
trigger:
- main   # or the branch you want
variables:
  kubernetesServiceConnection: 'aks-httpd-connection'  # <-- change to your service connection name
  namespace: 'httpd-app-ns'
  manifestsFolder: 'k8s'

stages:
- stage: Deploy
  displayName: Deploy httpd to AKS
  jobs:
  - job: DeployHttpd
    displayName: Deploy httpd app
    pool:
      vmImage: 'ubuntu-latest'

    steps:
      # (Optional) Show the repo content for debugging
      - script: |
          echo "Listing repo files"
          ls -R
        displayName: 'List files'

      - task: KubernetesManifest@1
        displayName: 'Deploy httpd manifest to AKS'
        inputs:
          action: 'deploy'
          kubernetesServiceConnection: $(kubernetesServiceConnection)
          namespace: $(namespace)
          manifests: |
            $(manifestsFolder)/manifest.yml
```
7. Click on **Commit**
8. Add a commit message, e.g.:
   - `Add HTTPD deployment manifest`
9. Click **Commit** (or **Commit to main**)
10. Similerly Create New file for `httpd-app.yaml`
11. Click **New** and select file, give name `httpd-app.yaml` and Click **create**
12. Provided  file content as mentioned below:
  
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: httpd-app-ns

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment
  namespace: httpd-app-ns
spec:
  replicas: 2
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
        - name: httpd
          image: httpd:2.4   # public Docker Hub image
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"

---
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
  namespace: httpd-app-ns
spec:
  type: LoadBalancer
  selector:
    app: httpd
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
```
13. Click on **Commit**
    
---

## 🔐 Step 4: Create Kubernetes Service Connection in Azure DevOps

This service connection allows the pipeline to talk to the AKS cluster.

1. In Azure DevOps, click on the **gear icon** (bottom left) → **Project settings**.
2. Under **Pipelines**, click **Service connections**.
3. Click **New service connection**.
4. Select **Kubernetes** and click **Next**.
5. Choose **Azure Resource Manager** if you want to select AKS from subscription.
6. Authenticate with your Azure subscription if prompted.
7. Select:
   - **Subscription**: Your Azure subscription
   - **Kubernetes cluster**: Your AKS cluster
   - **Namespace**
   - **Service Connection Name**:Give a name such as `aks-httpd-connection`
8. Check **Grant access permission to all pipelines**.
10. Click **Save**.

> 💡 Note this name; it must match the name used in your pipeline YAML.

---


## Step 5: Create the Pipeline in Azure DevOps

1. In Azure DevOps left menu, click **Pipelines → Pipelines**.
2. Click New pipeline.
3. Choose Azure Repos Git.
4. Select the repo you created: k8s-aks-httpd
5. Select **Existing Azure Pipelines YAML file** and browse to: `azure-pipeline.yaml`
6. Click Continue or Review to see the YAML.
7. Click Save and run.

🎬 This will start your first pipeline run.

---

## 📡Step 6: Monitor the Pipeline Run

1. After clicking Save and run, you’ll be taken to the pipeline run details.
2. Wait for the job DeployHttpd (or similar) to start.
3. Click on the job to see detailed logs.
4. Check these key tasks:
- Checkout: Should complete successfully.
- KubernetesManifest@1 Deploy: Should show Apply successful messages.
- Ensure the run finishes with Status: Succeeded.
5. If there is an error (for example, wrong service connection name or missing manifest.yml), read the error log and fix the YAML or config accordingly, then re-run.

---
## 🔍Step 7: Verify Deployment Using kubectl

🔑 Ensure kubectl is configured to talk to your AKS cluster. If not, use Azure CLI:
```
az aks get-credentials -g <ResourceGroupName> -n <ClusterName>
```
Run to list namespaces and ensure httpd-app exists:
```
kubectl -n httpd-app-ns get pod
```


