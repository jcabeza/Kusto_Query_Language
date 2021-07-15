# KQL Queries for Azure AppInsights

## Query for AppInsight Details
```
resources
| where type =~ 'microsoft.insights/components'
| extend RetentionInDays = properties.RetentionInDays
| extend IngestionMode = properties.IngestionMode
| extend Details = pack_all()
| project Resource=id, location, resourceGroup, subscriptionId, IngestionMode, RetentionInDays, Details
```
