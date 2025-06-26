

## 🧪 Lab Guide: Create Azure Resource Group (via GUI)
### ✅ Step 1: Log in to Azure Portal
* Go to: https://portal.azure.com
* Sign in with your Azure account.

### ✅ Step 2: Open Resource Groups Blade
* On the left-hand menu, click "Resource groups"
(If not visible, click the hamburger icon ≡ → Search for “Resource groups”).

* Click the “+ Create” button at the top.

### ✅ Step 3: Configure Resource Group
* Subscription: Select your active subscription.

* Resource Group Name: Example — K8S-RG

* Region: Choose a region, e.g., East US, Central India, etc.

### ✅ Step 4: Review and Create
* Click "Review + create"

* Click "Create"

✔ Done! Your resource group will be created in a few seconds.

### ✅ Step 5: Verify
G* o back to the Resource Groups page.

*  You should now see your new K8S-RG listed.

## 🧪 Lab Guide: Create AKS Cluster Using Azure Cloud Shell

🎯 Objective:
Use Azure Cloud Shell to create a Resource Group, deploy an AKS cluster, connect to it, and verify that it's working.

🛠️ Prerequisites:
An active Azure subscription

Browser access to https://portal.azure.com

###  ✅ Step 1: Open Azure Cloud Shell
Open your browser and go to 👉 https://portal.azure.com

* In the top-right corner, click the Cloud Shell icon (🖥️ >_).

* If prompted:
*   Select Bash.
*   Choose Create Storage if required (first-time users only).
*   Wait a few seconds for the shell to initialize.

### ✅ Step 2: Set Your Environment Variables
📘 These variables help you reuse values without typing them again.

👨‍💻 In Cloud Shell, type:
```
RESOURCE_GROUP="K8S-RG"
CLUSTER_NAME="aks-cluster"
LOCATION="centralus"
```
✅ You’ve now stored your desired group name, cluster name, and region in memory.

### ✅ Step 3: Create a Resource Group
📘 A resource group is a container that holds related Azure resources.

👨‍💻 Type the following command:
```
az group create --name $RESOURCE_GROUP --location $LOCATION
```

✅ Expected output:
A JSON response confirming that the resource group K8S-RG has been created.

### ✅ Step 4: Create the AKS Cluster
📘 Now, create the actual Kubernetes cluster with 2 worker nodes.

👨‍💻 Type:
```
az aks create \
  --resource-group $RESOURCE_GROUP \
  --name $CLUSTER_NAME \
  --node-count 2 \
  --node-vm-size Standard_DS2_v2 \
  --generate-ssh-keys
```
⏳ This takes about 3–7 minutes.

✅ You’ll see details like fqdn, kubeConfig, etc., once the cluster is created.

###  ✅ Step 5: Connect to the AKS Cluster
📘 You need to connect kubectl to your new cluster.

👨‍💻 Type:
```
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```
✅ You’ll see a message like:

Merged "aks-cluster" as current context in /home/<user>/.kube/config

### ✅ Step 6: Verify Your Cluster is Running
📘 This command lists all nodes in your AKS cluster.

👨‍💻 Type:
```
kubectl get nodes
```
✅ You should see 2 nodes with STATUS = Ready.
