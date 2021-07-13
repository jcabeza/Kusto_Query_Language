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
