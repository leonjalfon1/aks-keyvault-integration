# Flex Volume
https://github.com/Azure/kubernetes-keyvault-flexvol

## Overview

 - In this tutorial we will do the following:
   - Set up the environment (configure some environment variables)
   - Install Key Vault FlexVolume in your AKS cluster
   - Configure FlexVolume (create a secret with service principal details)
   - Deploy a pod that access to a Key Vault secret
   - Cleanup

## Set Up The Environment

 - Let's configure some environment variables that will be used during the tutorial

 ```
 KEY_VAULT_NAME=akskeyvaulttutorial
 KEY_VAULT_SECRET_NAME=mySecret
 SERVICE_PRINCIPAL_CLIENT_ID=<your-service-principal-client-id>
 SERVICE_PRINCIPAL_CLIENT_SECRET=<your-service-principal-secret>
 SERVICE_PRINCIPAL_TENANT_ID=<your-service-principal-tenant-id>
 ```

## Installation

 - Deploy Key Vault FlexVolume to your AKS cluster

 ```
 kubectl create -f https://raw.githubusercontent.com/Azure/kubernetes-keyvault-flexvol/master/deployment/kv-flexvol-installer.yaml
 ```

 - To validate Key Vault FlexVolume is running as expected, run the following command:
 ```
 kubectl get pods -n kv
 ```

 - The output should show keyvault-flexvolume pods running on each agent node:

 ```
 NAME                        READY     STATUS    RESTARTS   AGE
 keyvault-flexvolume-f7bx8   1/1       Running   0          3m
 keyvault-flexvolume-rcxbl   1/1       Running   0          3m
 keyvault-flexvolume-z6jm6   1/1       Running   0          3m
 ```

## Configuration

 - There are 4 options to configure FlexVolume: Service Principal, Pod identity, VMSS User Assigned Managed Identity or VMSS System Assigned Managed Identity (in this tutorial we will use a service principal due it's the simplest option)

 - Add your service principal credentials as Kubernetes secret

 ```
 kubectl create secret generic kvcreds --from-literal clientid=${SERVICE_PRINCIPAL_CLIENT_ID} --from-literal clientsecret=${SERVICE_PRINCIPAL_CLIENT_SECRET} --type=azure/kv
 ```

## Usage

 - Let's create a pod to test our configuration:

 ```
 cat << EOF | kubectl apply -f -
 apiVersion: v1
 kind: Pod
 metadata:
   name: flex-kv-test
 spec:
   containers:
   - name: flex-kv-test
     image: nginx
     volumeMounts:
     - name: test
       mountPath: /kvmnt
       readOnly: true
   volumes:
   - name: test
     flexVolume:
       driver: "azure/kv"
       secretRef:
         name: kvcreds
       options:
         usepodidentity: "false"                         # [OPTIONAL] if not provided, will default to "false"
         keyvaultname: "${KEY_VAULT_NAME}"               # the name of the KeyVault
         keyvaultobjectnames: ${KEY_VAULT_SECRET_NAME}   # list of KeyVault object names (semi-colon separated)
         keyvaultobjecttypes: secret                     # list of KeyVault object types: secret, key or cert (semi-colon separated)
         keyvaultobjectversions: ""                      # [OPTIONAL] list of KeyVault object versions (semi-colon separated), default:latest
         tenantid: "${SERVICE_PRINCIPAL_TENANT_ID}"      # the tenant ID of the KeyVault
 EOF
 ```

 - Ensure that you have access to keyvault from the deployed pod

 ```
 kubectl exec -it flex-kv-test cat /kvmnt/${KEY_VAULT_SECRET_NAME}
 ```

## Cleanup

 - Delete the test pod
 
 ```
 kubectl delete pod flex-kv-test
 ```

 - Delete the secret that store the service principal

 ```
 kubectl delete secret kvcreds
 ```

 - Uninstall FlexVolume from your AKS cluster 

 ```
 kubectl delete daemonset keyvault-flexvolume -n kv
 kubectl delete namespace kv
 ```