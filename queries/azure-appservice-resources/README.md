# KQL Queries for Azure App Service Resources

## Query App Service that are not functionapp
```
resources
| where type == "microsoft.web/sites" and kind notcontains "functionapp"
```

## Query App Services Kinds
```
resources
| where type =~'Microsoft.Web/Sites'
| summarize count(name) by kind
```
