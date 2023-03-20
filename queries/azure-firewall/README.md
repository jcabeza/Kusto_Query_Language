# KQL Queries for Azure Firewall

## Query for check Network Flow
```
AzureDiagnostics
| where TimeGenerated > ago(7h)  
| where Category == "AzureFirewallNetworkRule"  
| parse msg_s with Protocol " request from " SourceIP " to " Target ". Action: " Action  
| project TimeGenerated, Protocol, SourceIP, Target, Action, Complete_MSG=msg_s
```
```
AzureDiagnostics
| where TimeGenerated > ago(7h)
| where Category == "AzureFirewallApplicationRule"
| parse msg_s with Protocol " request from " SourceIP " to " Target ". Action: " Action
| project TimeGenerated, Protocol, SourceIP, Target, Action, Complete_MSG=msg_s
| where SourceIP == "192.168.1.1"
```

## Query for check Deny specific port
```
AzureDiagnostics
| where TimeGenerated > ago(7h)
| where Category  == 'AzureFirewallNetworkRule'
| where msg_s has_any ('Deny','UDP','53')
```

## Query for search in a range time
```
AzureDiagnostics
| where TimeGenerated between(datetime("2022-01-05 00:00:00") .. datetime("2022-01-08 12:00:00"))
| where Category == "AzureFirewallNetworkRule"
| parse msg_s with Protocol " request from " SourceIP " to " Target ". Action: " Action
| project TimeGenerated, SourceIP, Target, Action, msg_s
```
