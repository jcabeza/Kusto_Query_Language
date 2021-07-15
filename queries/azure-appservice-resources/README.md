# KQL Queries for Azure App Service Resources

## Query App Service that are not functionapp
```
resources
| where type == "microsoft.web/sites" and kind notcontains "functionapp"
```

## Query App Services Kinds
```
resources
| where type =~'Microsoft.Web/Sites'
| summarize count(name) by kind
```

## Query for Count App Services by Type
```
resources
| where type has 'microsoft.web'
           or type =~ 'microsoft.apimanagement/service'
           or type =~ 'microsoft.network/frontdoors'
           or type =~ 'microsoft.network/applicationgateways'
           or type =~ 'microsoft.appconfiguration/configurationstores'
| extend type = case(
           type == 'microsoft.web/serverfarms', "App Service Plans",
           kind == 'functionapp', "Azure Functions",
           kind == "api", "API Apps",
           type == 'microsoft.web/sites', "App Services",
           type =~ 'microsoft.network/applicationgateways', 'App Gateways',
type =~ 'microsoft.network/frontdoors', 'Front Door',
           type =~ 'microsoft.apimanagement/service', 'API Management',
           type =~ 'microsoft.web/certificates', 'App Certificates',
           type =~ 'microsoft.appconfiguration/configurationstores', 'App Config Stores',
           strcat("Not Translated: ", type))
| where type !has "Not Translated"
| summarize count() by type
```

## Query for App Service Full Details
```
resources
| where type has 'microsoft.web'
           or type =~ 'microsoft.apimanagement/service'
           or type =~ 'microsoft.network/frontdoors'
           or type =~ 'microsoft.network/applicationgateways'
           or type =~ 'microsoft.appconfiguration/configurationstores'
| extend type = case(
           type == 'microsoft.web/serverfarms', "App Service Plans",
           kind == 'functionapp', "Azure Functions",
           kind == "api", "API Apps",
           type == 'microsoft.web/sites', "App Services",
           type =~ 'microsoft.network/applicationgateways', 'App Gateways',
           type =~ 'microsoft.network/frontdoors', 'Front Door',
           type =~ 'microsoft.apimanagement/service', 'API Management',
           type =~ 'microsoft.web/certificates', 'App Certificates',
           type =~ 'microsoft.appconfiguration/configurationstores', 'App Config Stores',
           strcat("Not Translated: ", type))
| where type !has "Not Translated"
| extend Sku = case(
           type =~ 'App Gateways', properties.sku.name,
           type =~ 'Azure Functions', properties.sku,
           type =~ 'API Management', sku.name,
           type =~ 'App Service Plans', sku.name,
           type =~ 'App Services', properties.sku,
           type =~ 'App Config Stores', sku.name,
           ' ')
| extend State = case(
           type =~ 'App Config Stores', properties.provisioningState,
           type =~ 'App Service Plans', properties.status,
           type =~ 'Azure Functions', properties.enabled,
           type =~ 'App Services', properties.state,
           type =~ 'API Management', properties.provisioningState,
           type =~ 'App Gateways', properties.provisioningState,
           type =~ 'Front Door', properties.provisioningState,
           ' ')
| mv-expand publicIpId=properties.frontendIPConfigurations
| mv-expand publicIpId = publicIpId.properties.publicIPAddress.id
| extend publicIpId = tostring(publicIpId)
           | join kind=leftouter(
               Resources
               | where type =~ 'microsoft.network/publicipaddresses'
               | project publicIpId = id, publicIpAddress = tostring(properties.ipAddress))
               on publicIpId
| extend PublicIP = case(
           type =~ 'API Management', properties.publicIPAddresses,
           type =~ 'App Gateways', publicIpAddress,
           ' ')
| extend Details = pack_all()
| project Resource=id, type, subscriptionId, Sku, State, PublicIP, Details
```
