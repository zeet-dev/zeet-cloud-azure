# Azure Configuration

## Cert Manager

- https://cert-manager.io/docs/tutorials/getting-started-aks-letsencrypt/

### $AZURE_SUBSCRIPTION_ID

Can be defined as:

```
export AZURE_SUBSCRIPTION_ID=$(az account show --name <your-subscription-or-billing-account> --query 'id' -o tsv)
```

### $USER_ASSIGNED_IDENTITY_CLIENT_ID

```
export USER_ASSIGNED_IDENTITY_CLIENT_ID=$(az identity show --name "<managed-identity-name>" --query 'clientId' -o tsv)
az role assignment create \
    --role "DNS Zone Contributor" \
    --assignee $USER_ASSIGNED_IDENTITY_CLIENT_ID \
    --scope $(az network dns zone show --name $DOMAIN_NAME -o tsv --query id)
```

### $AZURE_ZONE_NAME: DNS domain name

Creating a public domain name in Azure

```
az network dns zone create --name $AZURE_ZONE_NAME
```

### $AZURE_RESOURCE_GROUP: The Azure resource group containing the DNS zone

## DNS

- https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/azure.md

### azure.json configuration

Copy to `/etc/kubernetes/azure.json`

#### $AZURE_TENENT_ID (required)

```
az account show --query "tenantId"
```

### $AZURE_SUBSCRIPTION_ID (required)

```
az account show --query "id"
```

### $AZURE_RESOURCE_GROUP (required)

Same of `$AZURE_RESOURCE_GROUP` from cert-manager