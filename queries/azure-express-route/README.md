# KQL Queries for Azure Express Route

## Query for IKE diagnostics
```
AzureDiagnostics 
| where Category == "IKEDiagnosticLog" 
| where TimeGenerated > ago(30m)
| sort by TimeGenerated asc
```

## Query for Tunnel diagnostics
```
AzureDiagnostics 
| where TimeGenerated > ago(24h)
| where Category == "TunnelDiagnosticLog"
//| project TimeGenerated, Resource , status_s, remoteIP_s, stateChangeReason_s
```