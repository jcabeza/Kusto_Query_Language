# KQL Queries for Azure Virtual Machines

## List of Virtual Machines
```
Resources
| where type =~ 'microsoft.compute/virtualmachines'
| project-away tenantId, kind, managedBy, sku, plan, tags, identity, zones, extendedLocation
```

## List of Virtual Machine and Size
```
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| project vmName = name, vmSize=tostring(properties.hardwareProfile.vmSize), vmId = id
| project-away vmId
```

## List of Virtual Machine and Current State
```
Resources 
| where type == "microsoft.compute/virtualmachines" 
| extend vmState = tostring(properties.extended.instanceView.powerState.displayStatus) 
| extend vmState = iif(isempty(vmState), "VM State Unknown", (vmState)) 
| summarize count() by vmState
```

```
Resources
| where type == 'microsoft.compute/virtualmachines'
| summarize count() by tostring(properties.extended.instanceView.powerState.code)
```

## Count Virtual Machines by OS type
```
Resources
| where type =~ 'Microsoft.Compute/virtualMachines'
| summarize count() by tostring(properties.storageProfile.osDisk.osType)
```
