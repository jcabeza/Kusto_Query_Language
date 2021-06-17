# Kusto Query Language (KQL)

## Getting started with Kusto
https://docs.microsoft.com/en-us/azure/data-explorer/kusto/concepts/

## KQL Quick Reference
https://docs.microsoft.com/en-us/azure/data-explorer/kql-quick-reference

## KQL Presentation
https://youtu.be/WBiMTLIJXMo

## KQL Cheat Sheet
[kql_cheat_sheet.pdf](https://github.com/jcabeza/Kusto_Query_Language/files/6668139/kql_cheat_sheet.pdf)

## KQL Queries

### Find Orphaned Disks
```
Resources
| where type has "microsoft.compute/disks"
| extend diskState = tostring(properties.diskState)
| where managedBy == "" or diskState == 'Unattached'
| project id, diskState, resourceGroup, location, subscriptionId
```

### Find Orphaned NICs
```
Resources
| where type has "microsoft.network/networkinterfaces"
| where "{nicWithPrivateEndpoints}" !has id
| where properties !has 'virtualmachine'
| project id, resourceGroup, location, subscriptionId
```

### Find Orphaned NSG (no nic or subnet)
```
Resources
| where type =~ 'microsoft.network/networksecuritygroups' and isnull(properties.networkInterfaces) and isnull(properties.subnets)
| project Resource=id, resourceGroup, subscriptionId, location
```

### Find Orphaned Public IP
```
resources 
| where type =~ 'microsoft.network/publicipaddresses' 
| extend IpConfig = properties.ipConfiguration.id 
| where isempty(IpConfig)
```
