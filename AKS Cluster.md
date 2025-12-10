# Lab: Create an Azure Kubernetes Service (AKS) Cluster (Portal)

## 1. Lab Overview

In this lab you will:

1. Create a **Resource Group**
2. Create an **AKS cluster** using the Azure Portal
3. Connect to the cluster using **Azure CLI + kubectl**
4. Verify that the cluster is working
5. (Optional) Clean up resources

> **Theory (very short)**  
> - **AKS** = Managed Kubernetes on Azure.  
> - **Resource Group** = Logical folder for Azure resources.  
> - **Node pool** = Set of VMs that run your pods.  
> - **Standard_D2s_v3** = Common VM size (2 vCPU, 8 GB RAM).

---

## 2. Prerequisites

- Azure subscription (you can use a free trial or student subscription).
- Azure CLI installed on your local machine or use **Azure Cloud Shell**.
- Basic understanding of Kubernetes terms: **cluster, node, pod**.

---

## 3. Task 1 – Sign in and Open AKS Creation Wizard

1. Go to [Azure Portal](https://portal.azure.com) and **Sign in**.
2. In the top search bar, type **“Kubernetes services”** and select it.
3. Click **+ Create** → **+ Create Kubernetes cluster**.

 
<img width="1402" height="854" alt="image" src="https://github.com/user-attachments/assets/ea8bb66c-2e19-4ee8-8838-7c34eb4e2a0b" />
    
   You should see a screen similar to the screenshot you shared.

---

## 4. Task 2 – Configure Basic Cluster Settings

You are on the **Basics** tab.

### 4.1 Project details

1. **Subscription**: choose your available subscription.  
2. **Resource group**  
   - Click **Create new**.  
   - Name: `rg-aks-lab` (or any name you like).  
   - Click **OK**.

### 4.2 Cluster details

1. **Cluster preset configuration**: select **Dev/Test**.
2. **Kubernetes cluster name**: `aks-lab-cluster` (or your own name).
3. **Region**: choose a region close to you, e.g. **(US) Central US** or **(Asia Pacific) Central India**.
4. **AKS pricing tier** (if visible):
   - Choose **Standard** (default).

<img width="1400" height="897" alt="image" src="https://github.com/user-attachments/assets/4904eedd-2c77-4e01-af53-998ce5e374f3" />


---

## 5. Task 3 – Configure Node Pool

1. **Node size**: click **Change size**.
   - Search for `Standard_D2s_v3`.
   - Select **Standard_D2s_v3**.
   - Click **Select**.
2. **Node count**:
   - Set **Node count** to `1` or `2` (for lab, `1` is enough and cheaper).
3. Leave other options at default (System node pool, Virtual Machine scale set).

Click **Next: Access** (or select **Access** from the top tabs).

---

## 6. Task 4 – Configure Access (Authentication & Authorization)

On the **Access** tab:

1. **Authentication method**: keep **System-assigned managed identity** (default).
2. **Local accounts with Kubernetes RBAC**:  
   - For a simple lab, select **Local accounts with Kubernetes RBAC**.
3. **Azure AD integration**: leave defaults (off) for now.

Click **Next: Networking**.

---

## 7. Task 5 – Configure Networking

On the **Networking** tab:

1. **Network configuration**:
   - Choose **Kubenet** (simple, default) OR **Azure CNI** (more advanced, uses VNet IPs).
   - For basic lab: keep **Kubenet**.
2. **DNS name prefix**:
   - Example: `aks-lab-dns`.
3. Leave other settings as default (new virtual network, default subnets, etc.).

Click **Next: Integrations** or **Next: Monitoring** depending on portal layout.

---

## 8. Task 6 – Monitoring and Tags

### 8.1 Monitoring

1. **Enable Container insights**:  
   - For lab, you can **Enable** or **Disable**.
   - If **Enable**, select a **Log Analytics workspace** or create a new one.

### 8.2 Tags (optional)

1. Add tags if you want to track cost, for example:  
   - `env = dev`  
   - `owner = your-name`

Click **Review + create**.

---

## 9. Task 7 – Review and Create Cluster

1. Azure will run **Validation**.
   - If there are errors, read the message and fix the field.
2. Once validation passes, click **Create**.
3. Deployment will start. When it finishes, you will see **“Your deployment is complete”**.

Click **Go to resource** to open the AKS cluster overview page.

---

## 10. Task 8 – Connect to the AKS Cluster

You can use **Cloud Shell** or your local terminal.

### 10.1 Open Cloud Shell

1. In the top-right of Azure Portal, click the **Cloud Shell** icon (`>`_).
2. Choose **Bash**.
3. Make sure you are in the right subscription:

   ```bash
   az account show
   # If wrong subscription:
   az account set --subscription "<your-subscription-name-or-id>"
