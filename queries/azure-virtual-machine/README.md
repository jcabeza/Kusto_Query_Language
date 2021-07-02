# KQL Queries for Azure Virtual Machines

## List of Virtual Machines
```
Resources
| where type =~ 'microsoft.compute/virtualmachines'
| project-away tenantId, kind, managedBy, sku, plan, tags, identity, zones, extendedLocation
```

```

```

```

```

```

```

```

```
