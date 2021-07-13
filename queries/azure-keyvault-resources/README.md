# KQL Queries for Azure Key Vault

## List of Key Vault
```
Resources
| where type =~ 'microsoft.keyvault/vaults'
| count
```
