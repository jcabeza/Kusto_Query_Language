

## Query for Count Enventing Services (Event Hub, EventGrid, Service Bus...)
```
resources
| where type has 'microsoft.servicebus'
           or type has 'microsoft.eventhub'
           or type has 'microsoft.eventgrid'
           or type has 'microsoft.relay'
| extend type = case(
           type == 'microsoft.eventgrid/systemtopics', "EventGrid System Topics",
           type =~ "microsoft.eventgrid/topics", "EventGrid Topics",
           type =~ 'microsoft.eventhub/namespaces', "EventHub Namespaces",
           type =~ 'microsoft.servicebus/namespaces', 'ServiceBus Namespaces',
           type =~ 'microsoft.relay/namespaces', 'Relays',
           strcat("Not Translated: ", type))
| where type !has "Not Translated"
| summarize count() by type
```

## Query for Enventing Services Full Details (Status, Endpoint...)
```
resources
| where type has 'microsoft.servicebus'
           or type has 'microsoft.eventhub'
           or type has 'microsoft.eventgrid'
           or type has 'microsoft.relay'
| extend type = case(
           type == 'microsoft.eventgrid/systemtopics', "EventGrid System Topics",
           type =~ "microsoft.eventgrid/topics", "EventGrid Topics",
           type =~ 'microsoft.eventhub/namespaces', "EventHub Namespaces",
           type =~ 'microsoft.servicebus/namespaces', 'ServiceBus Namespaces',
           type =~ 'microsoft.relay/namespaces', 'Relays',
           strcat("Not Translated: ", type))
| extend Sku = case(
           type =~ 'Relays', sku.name,
           type =~ 'EventGrid System Topics', properties.sku,
           type =~ 'EventGrid Topics', sku.name,
           type =~ 'EventHub Namespaces', sku.name,
           type =~ 'ServiceBus Namespaces', sku.sku,
           ' ')
| extend Endpoint = case(
           type =~ 'Relays', properties.serviceBusEndpoint,
           type =~ 'EventGrid Topics', properties.endpoint,
           type =~ 'EventHub Namespaces', properties.serviceBusEndpoint,
           type =~ 'ServiceBus Namespaces', properties.serviceBusEndpoint,
           ' ')
| extend Status = case(
           type =~ 'Relays', properties.provisioningState,
           type =~ 'EventGrid System Topics', properties.provisioningState,
           type =~ 'EventGrid Topics', properties.publicNetworkAccess,
           type =~ 'EventHub Namespaces', properties.status,
           type =~ 'ServiceBus Namespaces', properties.status,
           ' ')
| extend Details = pack_all()
| project Resource=id, subscriptionId, resourceGroup, Sku, Status, Endpoint, Details
```
