# cna-devops-03

Unfortunately I had to delete my resource group to avoid unecessary costs while learning but I wanted to
keep a record of what I did in the README. So moving forward the code will be deleted excluding the README.

# Step One
Created ArmTemplates with aks-template.json file to be used by the github action in the creation of
an Azure Kubernetes Service 

````
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "clusterName": {
            "type": "string"
        },
        "dnsPrefix": {
            "type": "string"
        },
        "clusterLocation": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        },
        "agentCount": {
            "defaultValue": 3,
            "type": "int"
        },
        "agentVMSize": {
            "defaultValue": "Standard_D2_v2",
            "type": "string"
        },
        "kubernetesVersion": {
            "type": "string"
        },
        "clusterTags": {
            "type": "object",
            "defaultValue": {}
        },
        "httpApplicationRoutingEnabled": {
            "type": "bool"
        }
    },
    "variables": {
        "clusterId": "[concat('Microsoft.ContainerService/managedClusters/',parameters('clusterName'))]"
    },
    "resources": [
        {
            "apiVersion": "2020-03-01",
            "type": "Microsoft.ContainerService/managedClusters",
            "location": "[parameters('clusterLocation')]",
            "name": "[parameters('clusterName')]",
            "tags": "[parameters('clusterTags')]",
            "dependsOn": [
            ],
            "properties": {
                "dnsPrefix": "[parameters('dnsPrefix')]",
                "kubernetesVersion": "[parameters('kubernetesVersion')]",
                "addonProfiles": {
                    "httpApplicationRouting": {
                        "enabled": "[parameters('httpApplicationRoutingEnabled')]"
                    }
                },
                "agentPoolProfiles": [
                    {
                        "name": "agentpool",
                        "count": "[parameters('agentCount')]",
                        "vmSize": "[parameters('agentVMSize')]"
                    }
                ]
            },
            "identity": {
                "type": "SystemAssigned"
            }
        }
    ],
    "outputs": {
        "applicationRoutingZone": {
            "value": "[if(parameters('httpApplicationRoutingEnabled'), reference(variables('clusterId')).addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName, '')]",
            "type": "string"
        }
    }
}

````

# STEP TWO
Used the Secrets functionality of github actions to embed my Azure credentials in the repo for use on the AKS template
by the github action

# STEP THREE
Created aks-deploy.yml and updated K8's version so the github action could create a AKS from the template and deploy it
to my Azure Profile in the correct Resource Group

````
name: Deploy AKS
on: 
  workflow_dispatch:

env:
  RESOURCEGROUPNAME: "cna-devops-03-rg"
  SUBSCRIPTIONID: "$SUBSCRIPTIONID"
  CLUSTERNAME: "cna-devops-03-aks"
  AGENTCOUNT: "3"
  AGENTVMSIZE: "Standard_DS2_v2"
  KUBERNETESVERSION: 1.25.5
  HTTPSAPPLICATIONROUTINGENABLED: false
  KUBERNETESAPI: "apps/v1"

jobs:
  deploy:
    name: Deploy AKS cluster
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2

    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Create AKS Cluster
      uses: azure/arm-deploy@main
      id: deploy
      with:
        scope: resourcegroup
        subscriptionId: ${{ env.SUBSCRIPTIONID }}
        resourceGroupName: ${{ env.RESOURCEGROUPNAME }}
        template: ./ArmTemplates/aks-template.json
        parameters: clusterName="${{ env.CLUSTERNAME }}" agentCount="${{ env.AGENTCOUNT }}" agentVMSize="${{ env.AGENTVMSIZE }}" kubernetesVersion="${{ env.KUBERNETESVERSION }}" httpApplicationRoutingEnabled="${{ env.HTTPSAPPLICATIONROUTINGENABLED }}"  dnsPrefix="${{ env.CLUSTERNAME }}"

````

# STEP FOUR
Tested that everything worked and deleted the RG to avoid costs!
## First Github action was successful! CI/CD Piplines watch out! 
