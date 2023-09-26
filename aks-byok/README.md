# Azure Configuration

### Azure Account

## $AZURE_SUBSCRIPTION_ID

Can be defined as:

```
export AZURE_SUBSCRIPTION_ID=$(az account show --name <your-subscription-or-billing-account> --query 'id' -o tsv)
```

#### $AZURE_TENANT_ID (required)

```
az account show --query "tenantId"
```

### $AZURE_SUBSCRIPTION_ID (required)

```
az account show --query "id"
```

### $AZURE_RESOURCE_GROUP (required)

The Azure resource group containing the AKS Cluster and DNS zone


## Networking Stack

### $CLUSTER_DOMAIN: DNS domain name

Creating a public domain name in Azure

```
az network dns zone create --name $CLUSTER_DOMAIN
```

### Cert Manager

- https://cert-manager.io/docs/tutorials/getting-started-aks-letsencrypt/


#### $DNS_MANAGER_IDENTITY_CLIENT_ID

```
export DNS_MANAGER_IDENTITY_NAME=${CLUSTER_NAME}-dns-manager
az identity create --name "${DNS_MANAGER_IDENTITY_NAME}" --resource-group $AZURE_RESOURCE_GROUP

export DNS_MANAGER_IDENTITY_CLIENT_ID=$(az identity show --name "${DNS_MANAGER_IDENTITY_NAME}" --query 'clientId' -o tsv --resource-group $AZURE_RESOURCE_GROUP)
export DNS_MANAGER_IDENTITY_PRINCIPAL_ID=$(az identity show --name "${DNS_MANAGER_IDENTITY_NAME}" --query 'principalId' -o tsv --resource-group $AZURE_RESOURCE_GROUP)
az role assignment create \
    --role "DNS Zone Contributor" \
    --assignee-object-id $DNS_MANAGER_IDENTITY_PRINCIPAL_ID \
    --assignee-principal-type ServicePrincipal \
    --scope $(az network dns zone show --name $CLUSTER_DOMAIN -o tsv --query id --resource-group $AZURE_RESOURCE_GROUP)
```

``````
export SERVICE_ACCOUNT_NAMESPACE=cert-manager
export SERVICE_ACCOUNT_NAME=cert-manager
export SERVICE_ACCOUNT_ISSUER=$(az aks show --resource-group $AZURE_RESOURCE_GROUP --name $CLUSTER_NAME --query "oidcIssuerProfile.issuerUrl" -o tsv)
az identity federated-credential create \
  --name "cert-manager" \
  --identity-name "${DNS_MANAGER_IDENTITY_NAME}" \
  --issuer "${SERVICE_ACCOUNT_ISSUER}" \
  --subject "system:serviceaccount:${SERVICE_ACCOUNT_NAMESPACE}:${SERVICE_ACCOUNT_NAME}" \
  --resource-group $AZURE_RESOURCE_GROUP
``````

## External DNS

- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/azure.md

```
RESOURCE_GROUP_ID=$(az group show --name "${AZURE_RESOURCE_GROUP}" --query "id" --output tsv)
az role assignment create --role "Reader" \
    --assignee-object-id $DNS_MANAGER_IDENTITY_PRINCIPAL_ID \
    --assignee-principal-type ServicePrincipal \
    --scope "${RESOURCE_GROUP_ID}"

export SERVICE_ACCOUNT_NAMESPACE=kube-system
export SERVICE_ACCOUNT_NAME=external-dns
az identity federated-credential create \
  --name "external-dns" \
  --identity-name "${DNS_MANAGER_IDENTITY_NAME}" \
  --issuer "${SERVICE_ACCOUNT_ISSUER}" \
  --subject "system:serviceaccount:${SERVICE_ACCOUNT_NAMESPACE}:${SERVICE_ACCOUNT_NAME}" \
  --resource-group $AZURE_RESOURCE_GROUP

```

