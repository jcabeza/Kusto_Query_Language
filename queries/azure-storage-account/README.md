# KQL Queries for Azure Storage Account

## List of Storage Account
```
Resources
| where type == "microsoft.storage/storageaccounts"
| project-away tenantId, managedBy, sku, identity, plan, zones, tags, properties, extendedLocation
```

## Search for a storage account by name
```
resources
| where type == "microsoft.storage/storageaccounts"
| where name == "storage-name"
```

## Query for discover Storage Account where HTTPS was not enabled, or where File or Blob encryption is disabled.
```
resources
| where type == "microsoft.storage/storageaccounts" 
| where aliases["Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly"] == "false" 
	or aliases["Microsoft.Storage/storageAccounts/enableFileEncryption"] == "false"
	or aliases["Microsoft.Storage/storageAccounts/enableBlobEncryption"] == "false"
| project name, kind,  resourceGroup, subscriptionId
```
