# KQL Queries for Azure Application Gateway

## WAF Checks

### Check Block in WAF Logs
```
AzureDiagnostics 
| where TimeGenerated > ago(7d)
| where Category == "ApplicationGatewayFirewallLog" 
| where action_s == "Blocked" | order by TimeGenerated
```
```
AzureDiagnostics 
| where TimeGenerated > ago(7d)
| where Category == "ApplicationGatewayFirewallLog" 
| where action_s == "Blocked" | order by TimeGenerated
| project Category, Resource, requestUri_s, Message, clientIp_s, action_s, details_message_s, details_file_s, hostname_s, policyScopeName_s
```
### View the Raw data in the firewall log
```
AzureDiagnostics 
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
```
### Matched/Blocked requests by IP
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by clientIp_s, bin(TimeGenerated, 1m)
| render timechart
```
### Matched/Blocked requests by URI
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by requestUri_s, bin(TimeGenerated, 1m)
| render timechart
```
### Top matched rules
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by ruleId_s, bin(TimeGenerated, 1m)
| where count_ > 10
| render timechart
```
### Top five matched rule groups
```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize Count=count() by details_file_s, action_s
| top 5 by Count desc
| render piechart
```

## Links

https://docs.microsoft.com/en-us/azure/application-gateway/log-analytics
