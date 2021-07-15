# KQL Queries for Azure Automation

## Count of Automation
```
resources
| where type has 'microsoft.automation'
| extend type = case(
           type =~ 'microsoft.automation/automationaccounts', 'Automation Accounts',
           type =~ 'microsoft.automation/automationaccounts/runbooks', 'Automation Runbooks',
           type =~ 'microsoft.automation/automationaccounts/configurations', 'Automation Configurations',
           strcat("Not Translated: ", type))
| summarize count() by type
```

## List of Automation, Runbook Type and State
```
resources
| where type has 'microsoft.automation'
| extend type = case(
           type =~ 'microsoft.automation/automationaccounts', 'Automation Accounts',
           type =~ 'microsoft.automation/automationaccounts/runbooks', 'Automation Runbooks',
           type =~ 'microsoft.automation/automationaccounts/configurations', 'Automation Configurations',
           strcat("Not Translated: ", type))
| extend RunbookType = tostring(properties.runbookType)
| extend LogicAppTrigger = properties.definition.triggers
| extend LogicAppTrigger = iif(type =~ 'LogicApps', case(
           LogicAppTrigger has 'manual', tostring(LogicAppTrigger.manual.type),
           LogicAppTrigger has 'Recurrence', tostring(LogicAppTrigger.Recurrence.type),
           strcat("Unknown Trigger type", LogicAppTrigger)), LogicAppTrigger)
| extend State = case(
           type =~ 'Automation Runbooks', properties.state,
           type =~ 'Automation Accounts', properties.state,
           type =~ 'Automation Configurations', properties.state,
           ' ')
| extend CreatedDate = case(
           type =~ 'Automation Runbooks', properties.creationTime,
           type =~ 'LogicApps', properties.createdTime,
           type =~ 'Automation Accounts', properties.creationTime,
           type =~ 'Automation Configurations', properties.creationTime,
           ' ')
| extend LastModified = case(
           type =~ 'Automation Runbooks', properties.lastModifiedTime,
           type =~ 'Automation Accounts', properties.lastModifiedTime,
           type =~ 'Automation Configurations', properties.lastModifiedTime,
           ' ')
| extend Details = pack_all()
| project subscriptionId, type, resourceGroup, RunbookType, State, Details
```
