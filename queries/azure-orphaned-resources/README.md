# KQL Queries for Azure Orphaned Resources

## Find Orphaned Disks
```
Resources
| where type has "microsoft.compute/disks"
| extend diskState = tostring(properties.diskState)
| where managedBy == "" or diskState == 'Unattached'
| project id, diskState, resourceGroup, location, subscriptionId
```
```
Resources
| where type =~ 'Microsoft.Compute/disks'
| where properties.diskState =~ 'Unattached'
| project-away tenantId, kind, managedBy, sku, plan, tags, identity, zones, extendedLocation, properties
```

## Find Orphaned NICs
```
Resources
| where type has "microsoft.network/networkinterfaces"
| where "{nicWithPrivateEndpoints}" !has id
| where properties !has 'virtualmachine'
| project id, resourceGroup, location, subscriptionId
```

## Find Orphaned NSG (no nic or subnet)
```
Resources
| where type =~ 'microsoft.network/networksecuritygroups' and isnull(properties.networkInterfaces) and isnull(properties.subnets)
| project Resource=id, resourceGroup, subscriptionId, location
```

## Find Orphaned Public IP
```
resources 
| where type =~ 'microsoft.network/publicipaddresses' 
| extend IpConfig = properties.ipConfiguration.id 
| where isempty(IpConfig)
```
