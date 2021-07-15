# KQL Queries for Azure Log Analytics

## Query for List Log Analytics Workspaces and Application Insights
```
resources
| where type =~ 'microsoft.operationalinsights/workspaces' or type =~ 'microsoft.insights/components'
| summarize count() by type
| extend type = case(
              type == 'microsoft.insights/components', "Application Insights",
              type == 'microsoft.operationalinsights/workspaces', "Log Analytics workspaces",
              strcat(type, type))
```
