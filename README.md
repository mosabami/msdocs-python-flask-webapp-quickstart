# Stage 1: Deploy a Simple Python (Flask) web app to AKS Landing Zone

This is the sample Flask application for the Azure Quickstart [Deploy a Python (Django or Flask) web app to Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/quickstart-python). It is part of a multi-step process to deploy the smartbrain app but can also be used as a standalone guide. Access the main repo here: https://github.com/mosabami/smartbrain. 

In this stage you will follow the instructions below to deploy it into an AKS Landing Zone. Get started by creating the landing zone using AKS Deploy Helper using the commands provided below (you don't have to click on any of the links).

If you need an Azure account, you can [create on for free](https://azure.microsoft.com/en-us/free/).

## Getting started - AKS Deploy Helper

We can leverage [AKS Deploy Helper](https://github.com/Azure/AKS-Construction) to quickly create a suitable environment with AKS cluster and Azure Applicaton Gateway.

Begin deployment by running the commands below in a bash terminal. You can always use Azure cloud shell. This will deploy the following resources for the AKS landing zone:
* AKS cluster
* Virtual network
* Application gateway with WAF
* Azure container registry
* Managed identity for AKS and App gateway
* Azure keyvault
* Log analytics workspace

To make the deployment more secure, you will only allow access to your cluster from your ip address. Get your ip address by typing the following in google: "Whats my ip" in windows or entering `hostname -I` in linux or Azure cloudshell.
 > :warning: if the commands to deploy the cluster below don't work (likely because the static release used is outdated),or you dont like this configuration, generate your own command by using the (AKS Construction Helper)[https://aks.ms/AKSLZA/aksconstructionhelper]. Please note that some configurations wont work for the app deployment steps below without modification if you use a different configuration from the one below(eg if you decide to use a private cluster).

```bash
REGION=< your region, eg eastus>
RGNAME=smartbrain
```
```azurecli
az group create -l $REGION -n $RGNAME 
```

Deploy template with in-line parameters. 

> :warning: You might want to change the --template-url in the line below to the latest version available here: https://aka.ms/AKSLZA/aksconstructionhelper

```bash
az group create -l $REGION -n $RGNAME

az deployment group create -g $RGNAME  --template-uri https://github.com/Azure/AKS-Construction/releases/download/0.8.9/main.json --parameters \
	resourceName=aks-smartbrain \
	upgradeChannel=stable \
	agentCountMax=20 \
	custom_vnet=true \
	enable_aad=true \
	AksDisableLocalAccounts=true \
	enableAzureRBAC=true \
	adminPrincipalId=$(az ad signed-in-user show --query id --out tsv) \
	registries_sku=Premium \
	acrPushRolePrincipalId=$(az ad signed-in-user show --query id --out tsv) \
	omsagent=true \
	retentionInDays=30 \
	networkPolicy=azure \
	azurepolicy=audit \
	authorizedIPRanges="[\"<your ip address>\"]" \
	ingressApplicationGateway=true \
	appGWcount=0 \
	appGWsku=WAF_v2 \
	appGWmaxCount=10 \
	appgwKVIntegration=true \
	keyVaultAksCSI=true \
	keyVaultCreate=true \
	keyVaultOfficerRolePrincipalId=$(az ad signed-in-user show --query id --out tsv)
```

## Deploy the application
After cluster creation we can install the application onto the cluster

```bash
az aks get-credentials -g smartbrain -n aks-smartbrain --overwrite-existing
git clone https://github.com/mosabami/smartbrain
```

## Deploy the workload
Build the image
```bash
cd ./smartbrain/simpleapp
az acr build -t simpleapp:v1 -r $ACRNAME .
```
Switch to the k8s folder and update the k8s manifest file with your correct container registry name
```bash
cd k8s
code app-deployment.yaml
```
Deploy the workload
```bash
kubectl apply -f .
```
Get the ip address of the ingress controller and enter it in a browser to access the workload
```bash
kubectl get ingress 
```
Congratulations! you have your first app deployed in an AKS Landing Zone. Delete the workload
```bash
kubectl delete -f . 
cd ../..
```
## Next Step

If you are following the instructions in the Smartbrain repo, return to the next step there by clicking on this link below:

:arrow_forward: [Deploy and Smartbrain app](https://github.com/mosabami/smartbrain/tree/main/smartbrain)
