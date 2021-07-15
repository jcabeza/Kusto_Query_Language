# KQL Queries for Azure Databases

## Query for Count Databases (MySQL, CosmoDB, SQL Servers, Azure SQL DB)
```
resources
| where type =~ 'microsoft.documentdb/databaseaccounts'
        or type =~ 'microsoft.sql/servers/databases'
        or type =~ 'microsoft.dbformysql/servers'
        or type =~ 'microsoft.sql/servers'
| extend type = case(
              type =~ 'microsoft.documentdb/databaseaccounts', 'CosmosDB',
              type =~ 'microsoft.sql/servers/databases', 'SQL DBs',
              type =~ 'microsoft.dbformysql/servers', 'MySQL',
              type =~ 'microsoft.sql/servers', 'SQL Servers',
              strcat("Not Translated: ", type))
| where type !has "Not Translated"
| summarize count() by type
```

## Query for Databases Full Details (Type, SKU, Status, Public Endpoint, GB Size)
```
resources
| where type =~ 'microsoft.documentdb/databaseaccounts'
         or type =~ 'microsoft.sql/servers/databases'
         or type =~ 'microsoft.dbformysql/servers'
         or type =~ 'microsoft.sql/servers'
| extend type = case(
             type =~ 'microsoft.documentdb/databaseaccounts', 'CosmosDB',
             type =~ 'microsoft.sql/servers/databases', 'SQL DBs',
             type =~ 'microsoft.dbformysql/servers', 'MySQL',
             type =~ 'microsoft.sql/servers', 'SQL Servers',
             strcat("Not Translated: ", type))
| extend Sku = case(
             type =~ 'CosmosDB', properties.databaseAccountOfferType,
             type =~ 'SQL DBs', sku.name,
             type =~ 'MySQL', sku.name,
             ' ')
| extend Status = case(
             type =~ 'CosmosDB', properties.provisioningState,
             type =~ 'SQL DBs', properties.status,
             type =~ 'MySQL', properties.userVisibleState,
             ' ')
| extend Endpoint = case(
             type =~ 'MySQL', properties.fullyQualifiedDomainName,
             type =~ 'SQL Servers', properties.fullyQualifiedDomainName,
             type =~ 'CosmosDB', properties.documentEndpoint,
             ' ')
| extend maxSizeGB = todouble(case(
             type =~ 'SQL DBs', properties.maxSizeBytes,
             type =~ 'MySQL', properties.storageProfile.storageMB,
             ' '))
| extend maxSizeGB = iif(type has 'SQL DBs', maxSizeGB /1000 /1000, maxSizeGB)
| extend Details = pack_all()
| project Resource=id, resourceGroup, subscriptionId, type, Sku, Status, Endpoint, maxSizeGB, Details
```
