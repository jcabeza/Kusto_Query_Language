# KQL Queries for Azure Storage Account

## List of Storage Account
```
Resources
| where type == "microsoft.storage/storageaccounts"
| project-away tenantId, managedBy, sku, identity, plan, zones, tags, properties, extendedLocation
```

```

```

```

```

```

```
