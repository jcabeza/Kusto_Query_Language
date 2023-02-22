# KQL Queries for Azure Firewall

## Query for check Network Flow
```
AzureDiagnostics
| where TimeGenerated > ago(12h)  
| where Category == "AzureFirewallNetworkRule"  
| parse msg_s with Protocol " request from " SourceIP " to " Target ". Action: " Action  
| project TimeGenerated, Protocol, SourceIP, Target, Action, Complete_MSG=msg_s
```

```
AzureDiagnostics
| where TimeGenerated > ago(2h)
| where Category == "AzureFirewallApplicationRule"
| parse msg_s with Protocol " request from " SourceIP " to " Target ". Action: " Action
| project TimeGenerated, Protocol, SourceIP, Target, Action, Complete_MSG=msg_s
| where SourceIP == "192.168.1.1"
```
