# Commands

## Cluster

```sh
az aks create -n TripCluster -g teamResources --max-pods 100 --node-count 2 --enable-aad --enable-azure-rbac --network-plugin azure --vnet-subnet-id /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourceGroups/teamResources/providers/Microsoft.Network/virtualNetworks/vnet/subnets/vm-subnet --generate-ssh-keys --uptime-sla

az aks update -n TripCluster -g teamResources --attach-acr /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourceGroups/teamResources/providers/Microsoft.ContainerRegistry/registries/registryntt1010/subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourceGroups/teamResources/providers/Microsoft.ContainerRegistry/registries/registryntt1010

az aks update -n TripCluster -g teamResources --enable-aad --enable-azure-rbac

az aks get-credentials --name TripCluster --resource-group teamResources --overwrite-existing

```

## RBAC

```sh
az aks show --resource-group teamResources --name TripCluster --query id -o tsv
# /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourcegroups/teamResources/providers/Microsoft.ContainerService/managedClusters/TripCluster
az ad group create --display-name api-dev --mail-nickname api-dev --query objectId -o tsv
# 4700bbe2-84fd-402e-93e6-4482eb0d9337
az ad group create --display-name web-dev --mail-nickname web-dev --query objectId -o tsv
# e3c335eb-a16b-42d4-bee9-966d1c3840cf
az role assignment create --assignee 4700bbe2-84fd-402e-93e6-4482eb0d9337 --role "Azure Kubernetes Service Cluster User Role" --scope /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourcegroups/teamResources/providers/Microsoft.ContainerService/managedClusters/TripCluster

az role assignment create --assignee b1b12726-7404-4cfa-b4d0-04910a0a30b8 --role "Azure Kubernetes Service Cluster User Role" --scope /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourcegroups/teamResources/providers/Microsoft.ContainerService/managedClusters/TripCluster

# /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourcegroups/teamResources/providers/Microsoft.ContainerService/managedClusters/TripCluster/providers/Microsoft.Authorization/roleAssignments/fc2ea80d-0452-408c-9ee7-06fa39c7030e"
az role assignment create --assignee e3c335eb-a16b-42d4-bee9-966d1c3840cf --role "Azure Kubernetes Service Cluster User Role" --scope /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourcegroups/teamResources/providers/Microsoft.ContainerService/managedClusters/TripCluster
# add users to role
az ad user show --id "apidev@msftopenhack7003ops.onmicrosoft.com" --query "objectId" --output tsv
# b1b12726-7404-4cfa-b4d0-04910a0a30b8
az ad group member add --group 4700bbe2-84fd-402e-93e6-4482eb0d9337 --member-id b1b12726-7404-4cfa-b4d0-04910a0a30b8
# add users to role
az ad user show --id "" --query "objectId" --output tsv
# 734eb73b-acda-47c2-b61a-02e8245dbd15
az ad group member add --group e3c335eb-a16b-42d4-bee9-966d1c3840cf --member-id 734eb73b-acda-47c2-b61a-02e8245dbd15
```

```sh
kubectl auth can-i create deployments --namespace frontend --as api-dev
```

## KeyVault

```sh
az keyvault create --name kv-tripcluster-dev --resource-group teamResources --location NorthEurope
az keyvault secret set --name sql-username --vault-name kv-tripcluster-dev --value sqladminnTt1010
az keyvault secret set --name sql-password --vault-name kv-tripcluster-dev --value nV9zl6Qp9

az aks enable-addons --addons azure-keyvault-secrets-provider --name TripCluster --resource-group teamResources
az aks update -g teamResources -n TripCluster --enable-managed-identity

az aks show -g teamResources -n TripCluster --query addonProfiles.azureKeyvaultSecretsProvider.identity.clientId -o tsv
# 14222f08-8910-4a4a-9eba-78bf6f3b47b6
az ad sp show --id 14222f08-8910-4a4a-9eba-78bf6f3b47b6
# bda73996-36c5-48bd-bb19-64c3e72145fb
az role assignment create --role Reader --assignee 14222f08-8910-4a4a-9eba-78bf6f3b47b6 --scope /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourceGroups/teamResources
az role assignment create --role Managed Identity Operator --assignee 14222f08-8910-4a4a-9eba-78bf6f3b47b6 --scope /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourceGroups/teamResources
az role assignment create --role Virtual Machine Contributor --assignee 14222f08-8910-4a4a-9eba-78bf6f3b47b6 --scope /subscriptions/d47ffa85-a7bf-48fd-8d95-ccd2cbf5c42a/resourceGroups/teamResources


az keyvault set-policy -n kv-tripcluster-dev --key-permissions get --spn 14222f08-8910-4a4a-9eba-78bf6f3b47b6
az keyvault set-policy -n kv-tripcluster-dev --secret-permissions get list --spn 14222f08-8910-4a4a-9eba-78bf6f3b47b6
az keyvault set-policy -n kv-tripcluster-dev --certificate-permissions get --spn 14222f08-8910-4a4a-9eba-78bf6f3b47b6



az aks enable-addons -a monitoring -n TripCluster -g teamResources

az aks update -g teamResources -n TripCluster --enable-pod-identity --enable-pod-identity-with-kubenet # TODO: set policy

az aks enable-addons --addons azure-policy --name TripCluster --resource-group teamResources

az aks show --query addonProfiles.azurepolicy -g teamResources -n TripCluster

az policy remediation create --name myRemediation --policy-assignment '/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyAssignments/{myAssignmentId}'
/providers/Microsoft.Authorization/policyDefinitions/95edb821-ddaf-4404-9732-666045e056b4

kubectl get pods -n kube-system
kubectl get pods -n gatekeeper-system
az aks show --query addonProfiles.azurepolicy -g teamResources -n TripCluster

```
