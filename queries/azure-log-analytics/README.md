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

## Query for List Log Analytics Installed Solutions
```
resources
| where type == "microsoft.operationsmanagement/solutions"
| project Solution=plan.name, Workspace=tolower(tostring(properties.workspaceResourceId)), subscriptionId
| join kind=leftouter(
resources
| where type =~ 'microsoft.operationalinsights/workspaces'
| project Workspace=tolower(tostring(id)),subscriptionId) on Workspace
| summarize Solutions = strcat_array (make_list(Solution), ",") by Workspace, subscriptionId
| extend AzureSecurityCenter = iif(Solutions has 'Security','Enabled','Not Enabled')
| extend AzureSecurityCenterFree = iif(Solutions has 'SecurityCenterFree','Enabled','Not Enabled')
| extend AzureSentinel = iif(Solutions has "SecurityInsights",'Enabled','Not Enabled')
| extend AzureMonitorVMs = iif(Solutions has "VMInsights",'Enabled','Not Enabled')
| extend ServiceDesk = iif(Solutions has "ITSM Connector",'Enabled','Not Enabled')
| extend AzureAutomation = iif(Solutions has "AzureAutomation",'Enabled','Not Enabled')
| extend ChangeTracking = iif(Solutions has 'ChangeTracking','Enabled','Not Enabled')
| extend UpdateManagement = iif(Solutions has 'Updates','Enabled','Not Enabled')
| extend UpdateCompliance = iif(Solutions has 'WaaSUpdateInsights','Enabled','Not Enabled')
| extend AzureMonitorContainers = iif(Solutions has 'ContainerInsights','Enabled','Not Enabled')
| extend KeyVaultAnalytics = iif(Solutions has 'KeyVaultAnalytics','Enabled','Not Enabled')
| extend SQLHealthCheck = iif(Solutions has 'SQLAssessment','Enabled','Not Enabled')
```
