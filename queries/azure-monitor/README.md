#

## Query for Azure Monitor and Alerting (alerts, action groups, workbooks, dashboards...)
```
resources
| where type has 'microsoft.insights/'
        or type has 'microsoft.alertsmanagement/ smartdetectoralertrules'
        or type has 'microsoft.portal/dashboards'
| where type != 'microsoft.insights/components'
| extend type = case(
              type == 'microsoft.insights/workbooks', "Workbooks",
              type == 'microsoft.insights/activitylogalerts', "Activity Log Alerts",
              type == 'microsoft.insights/scheduledqueryrules', "Log Search Alerts",
              type == 'microsoft.insights/actiongroups', "Action Groups",
              type == 'microsoft.insights/metricalerts', "Metric Alerts",
              type =~ 'microsoft.alertsmanagement/smartdetectoralertrules','Smart Detection Rules',
              type =~ 'microsoft.insights/webtests', 'URL Web Tests',
              type =~ 'microsoft.portal/dashboards', 'Portal Dashboards',
              type =~ 'microsoft.insights/datacollectionrules', 'Data Collection Rules',
strcat("Not Translated: ", type))
| summarize count() by type
```

## Query for Azure Monitor Active Alerts
```
AlertsManagementResources
| extend AlertStatus = properties.essentials.monitorCondition
| extend AlertState = properties.essentials.alertState
| extend AlertTime = properties.essentials.startDateTime
| extend AlertSuppressed = properties.essentials.actionStatus.isSuppressed
| extend Severity = properties.essentials.severity
| where AlertStatus == 'Fired'
| extend Details = pack_all()
| project id, name, subscriptionId, resourceGroup, AlertStatus, AlertState, AlertTime, AlertSuppressed, Severity, Details
```

## Query for Azure Monitor Full Details
```
resources
| where type has 'microsoft.insights/'
or type has 'microsoft.alertsmanagement/smartdetectoralertrules'
or type has 'microsoft.portal/dashboards'
| where type != 'microsoft.insights/components' | extend type = case(
type == 'microsoft.insights/workbooks', "Workbooks",
type == 'microsoft.insights/activitylogalerts', "Activity Log Alerts",
type == 'microsoft.insights/scheduledqueryrules', "Log Search Alerts",
type == 'microsoft.insights/actiongroups', "Action Groups",
type == 'microsoft.insights/metricalerts', "Metric Alerts",
type =~ 'microsoft.alertsmanagement/smartdetectoralertrules','Smart Detection Rules',
type =~ 'microsoft.portal/dashboards', 'Portal Dashboards',
strcat("Not Translated: ", type))
| extend Enabled = case(
type =~ 'Smart Detection Rules', properties.state,
type != 'Smart Detection Rules', properties.enabled,
strcat("Not Translated: ", type))
| extend WorkbookType = iif(type =~ 'Workbooks', properties.category, ' ')
| extend Details = pack_all()
| project name, type, subscriptionId, location, resourceGroup, Enabled, WorkbookType, Details
```
