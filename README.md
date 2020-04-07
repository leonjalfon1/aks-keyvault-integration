# AKS and Key Vault Integration

## Description

In this tutorial we will explain how to integrate aks with azure key vault using ["FlexVolumes"](https://github.com/Azure/kubernetes-keyvault-flexvol) and ["Azure Key Vault to Kubernetes"](https://github.com/SparebankenVest/azure-key-vault-to-kubernetes)

## Overview

 - With **FlexVolumes** Key Vault secrets, keys, and certificates become a volume accessible to pods. Once the volume is mounted, its data is available directly in the container filesystem for your application.

 - **Azure Key Vault to Kubernetes** (akv2k8s) use two main components (Azure Key Vault Controller and Azure Key Vault Env Injector) to inject a secret, key or certificate as environment variable accessible only for the main process of the container.

|             | Pros                                                                                                     | Cons                                                                                                                                  |
|-------------|----------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| FlexVolumes | Easy to use and configure                                                                                | Anyone inside the container can see the secrets and the secrets are available only from the file system (not as environment variable) |
| akv2k8s     | Only the main process of the container can access the secrets and it's available as environment variable | More complicated infrastructure that requires a kubernetes CRD and a Mutating Admission Webhook                                                                        |

## Prerequisites

 - AKS Cluster (v0.0.14 or later)
 - Key Vault (including a secret)
 - Service Principal with "get" Access to Key Vault

---

 ### Prerequisites Configuration

 - Configure your environment (env variables to use in the next steps)

 ```
 SERVICE_PRINCIPAL_NAME=aks-keyvault-tutorial
 RESOURCE_GROUP_NAME=aks-keyvault-tutorial
 AKS_CLUSTER_NAME=aks-keyvault-tutorial
 KEY_VAULT_NAME=akskeyvaulttutorial   # Must be globally unique
 KEY_VAULT_SECRET_NAME=mySecret
 KEY_VAULT_SECRET_VALUE=myValue
 AZURE_LOCATION=westeurope
 ```

 - Install and configure Azure CLI

 ```
 https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest
 ```

- Create a Service Principal

 ```
 az ad sp create-for-rbac --name ${SERVICE_PRINCIPAL_NAME}
 ```

 - Create a Resource Group

 ```
 az group create --name ${RESOURCE_GROUP_NAME} --location ${AZURE_LOCATION}
 ```

 - Create an AKS cluster

 ```
 az aks create --resource-group ${RESOURCE_GROUP_NAME} --name ${AKS_CLUSTER_NAME} --node-count 1
 ```

 - Create a Key Vault
 
 ```
 az keyvault create -n ${KEY_VAULT_NAME} -g ${RESOURCE_GROUP_NAME}
 ```

 - Create a Key Vault Secret

 ```
 az keyvault secret set --vault-name ${KEY_VAULT_NAME} --name ${KEY_VAULT_SECRET_NAME} --value ${KEY_VAULT_SECRET_VALUE}
 ```

 - Authorize Access to Secrets for your service principal:

 ```
 az keyvault set-policy --n ${KEY_VAULT_NAME} --spn ${SERVICE_PRINCIPAL_NAME} --secret-permissions get 
 ```

 - Connect to the cluster

 ```
 az aks get-credentials --resource-group ${RESOURCE_GROUP_NAME} --name ${AKS_CLUSTER_NAME}
 ```

 - Test connection

 ```
 kubectl get nodes
 ```

## Content

 - [Integrate AKS with Key Vault using "FlexVolumes"](/flexvolumes/README.md)
 - [Integrate AKS with Key Vault using "Azure Key Vault to Kubernetes"](/akv2k8s/README.md)
