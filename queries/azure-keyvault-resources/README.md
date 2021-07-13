# KQL Queries for Azure Key Vault

## Query Key Vault Count
```
Resources
| where type =~ 'microsoft.keyvault/vaults'
| count
```

## List of Key Vault and Subscription Name
```
Resources
| join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| where type == 'microsoft.keyvault/vaults'
| project type, name, SubName
```
