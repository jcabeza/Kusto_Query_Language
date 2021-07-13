# KQL Queries for Azure Network Resources

## Query Specific PublicIP
```
Resources
| where type contains 'publicIPAddresses' and properties.ipAddress == "53.xx.xx.xx"
```

## Query Load Balancers With Standard SKU
```
resources
| where type == "microsoft.network/loadbalancers" and sku.name == "Standard"
```

## Query Virtual Network Gateways that are ExpressRoute Type
```
resources
| where type == "microsoft.network/virtualnetworkgateways" and properties.gatewayType == "ExpressRoute"
```

## Query Network Connections that are ExpressRoute Type
```
resources
| where type == "microsoft.network/connections" and properties.connectionType == "ExpressRoute"
```

## Query Network Security Groups expanding securityRules
```
Resources
| where type =~ "microsoft.network/networksecuritygroups"
| join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubcriptionName=name, subscriptionId) on subscriptionId
| mv-expand rules=properties.securityRules
| extend direction = tostring(rules.properties.direction)
| extend priority = toint(rules.properties.priority)
| extend access = rules.properties.access
| extend description = rules.properties.description
| extend destprefix = rules.properties.destinationAddressPrefix
| extend destport = rules.properties.destinationPortRange
| extend sourceprefix = rules.properties.sourceAddressPrefix
| extend sourceport = rules.properties.sourcePortRange
| extend subnet_name = split((split(tostring(properties.subnets), '/'))[10], '"')[0]
| project SubcriptionName, resourceGroup, subnet_name, name, direction, access, priority, sourceport, sourceprefix, destport, destprefix, description
| sort by SubcriptionName, resourceGroup asc, name, direction asc, priority asc
```
