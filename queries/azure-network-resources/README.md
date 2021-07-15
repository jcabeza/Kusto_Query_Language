# KQL Queries for Azure Network Resources

## Query Specific PublicIP
```
Resources
| where type contains 'publicIPAddresses' and properties.ipAddress == "53.xx.xx.xx"
```

## Query List Publics IP and Type
```
Resources
| where type == "microsoft.network/publicipaddresses"
| summarize PIPs=count() by IPType=tostring(properties.publicIPAddressVersion)
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

## Query List of Unassociated NSGs
```
Resources
| where type =~ 'microsoft.network/networksecuritygroups' and isnull(properties.networkInterfaces) and isnull(properties.subnets)
| project Resource=id, resourceGroup, subscriptionId, location
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
```
Resources
| where type =~ 'microsoft.network/networksecuritygroups'
| project id, nsgRules = parse_json(parse_json(properties).securityRules), networksecurityGroupName = name, subscriptionId, resourceGroup , location
| mvexpand nsgRule = nsgRules
| project id, location, access=nsgRule.properties.access,protocol=nsgRule.properties.protocol ,direction=nsgRule.properties.direction,provisioningState= nsgRule.properties.provisioningState ,priority=nsgRule.properties.priority,
                sourceAddressPrefix = nsgRule.properties.sourceAddressPrefix,
                sourceAddressPrefixes = nsgRule.properties.sourceAddressPrefixes,
                destinationAddressPrefix = nsgRule.properties.destinationAddressPrefix,
                destinationAddressPrefixes = nsgRule.properties.destinationAddressPrefixes,
                networksecurityGroupName, networksecurityRuleName = tostring(nsgRule.name),
                subscriptionId, resourceGroup,
                destinationPortRanges = nsgRule.properties.destinationPortRanges,
                destinationPortRange = nsgRule.properties.destinationPortRange,
                sourcePortRanges = nsgRule.properties.sourcePortRanges,
                sourcePortRange = nsgRule.properties.sourcePortRange
| extend Details = pack_all()
| project id, location, access, direction, subscriptionId, resourceGroup, Details
```

## Query All Network Resources
```
resources
| where type has "microsoft.network"
| extend type = case(
              type == 'microsoft.network/networkinterfaces', "NICs",
              type == 'microsoft.network/networksecuritygroups', "NSGs",
              type == "microsoft.network/publicipaddresses", "Public IPs",
              type == 'microsoft.network/virtualnetworks', "vNets",
              type == 'microsoft.network/networkwatchers/connectionmonitors', "Connection Monitors",
              type == 'microsoft.network/privatednszones', "Private DNS",
              type == 'microsoft.network/virtualnetworkgateways', @"vNet Gateways",
              type == 'microsoft.network/connections', "Connections",
              type == 'microsoft.network/networkwatchers', "Network Watchers",
              type == 'microsoft.network/privateendpoints', "Private Endpoints",
              type == 'microsoft.network/localnetworkgateways', "Local Network Gateways",
              type == 'microsoft.network/privatednszones/virtualnetworklinks', "vNet Links",
              type == 'microsoft.network/dnszones', 'DNS Zones',
              type == 'microsoft.network/networkwatchers/flowlogs', 'Flow Logs',
              type == 'microsoft.network/routetables', 'Route Tables',
              type == 'microsoft.network/loadbalancers', 'Load Balancers',
              strcat("Not Translated: ", type))
| summarize count() by type
| where type !has "Not Translated"
```
