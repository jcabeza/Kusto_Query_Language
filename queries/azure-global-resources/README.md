# KQL Queries for Azure Global Resources

## Count All Azure Resources and Ordering By Count
```
Resources
| summarize count() by type 
| order by count_
```

## Query and Count Resources Group Missing Tag
```
resourcecontainers
| where type == "microsoft.resources/subscriptions/resourcegroups"
| where tags !contains 'MyMissingTag'
| project name, resourceGroup, subscriptionId, location, tags
| summarize count () by subscriptionId
```

## Query All Resources Tags
```
resourcecontainers
| where type == "microsoft.resources/subscriptions/resourcegroups"
| project  name,type,location,subscriptionId,tags
| union (resources | project name,type,location,subscriptionId,tags)
```
