# KQL Queries for Azure Advisor

## Get cost savings summary from Azure Advisor
```
AdvisorResources
| where type == 'microsoft.advisor/recommendations'
| where properties.category == 'Cost'
| extend
	resources = tostring(properties.resourceMetadata.resourceId),
	savings = todouble(properties.extendedProperties.savingsAmount),
	solution = tostring(properties.shortDescription.solution),
	currency = tostring(properties.extendedProperties.savingsCurrency)
| summarize
	dcount(resources),
	bin(sum(savings), 0.01)
	by solution, currency
| project solution, dcount_resources, sum_savings, currency
| order by sum_savings desc
```
