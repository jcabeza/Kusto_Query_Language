# KQL Queries for Azure Global Resources

## Count All Azure Resources and Ordering By Count
```
Resources
| summarize count() by type 
| order by count_
```

```
// top ten resource types by number of resources
summarize ResourceCount=count() by type
| order by ResourceCount desc
| take 10
| project ["Resource Type"]=type, ["Resource Count"]=ResourceCount
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

## Query Azure Policy Non Compliant
```
policyresources
| where properties['policyDefinitionAction'] != "deny"
| where properties['complianceState'] == "NonCompliant"
| where properties['resourceGroup'] != ""
| project properties['resourceId'], resourceGroup, properties['policyDefinitionName'], properties['policyDefinitionReferenceId'], properties['complianceState']
| order by 'resourceId' asc
```
