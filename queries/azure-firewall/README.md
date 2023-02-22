# KQL Queries for Azure Firewall

## Query for check Network Flow
```
AzureDiagnostics
| where TimeGenerated > ago(12h)  
| where Category == "AzureFirewallNetworkRule"  
| parse msg_s with Protocol " request from " SourceIP " to " Target ". Action: " Action  
| project TimeGenerated, Protocol, SourceIP, Target, Action, Complete_MSG=msg_s
```
