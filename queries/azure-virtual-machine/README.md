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

## Count Virtual Machines by Size
```
Resources
| where type == "microsoft.compute/virtualmachines"
| summarize Count=count(properties.hardwareProfile.vmSize) by vmSize=tostring(properties.hardwareProfile.vmSize)
```

## Count Disk Size and Total GB
```
Resources
| where type contains "microsoft.compute/disks"
| summarize DiskSizeGB=sum(toint(properties.diskSizeGB))
```

## List Disk Size and Sku by Location
```
Resources
| where type contains "microsoft.compute/disks"
| summarize DiskSizeGB=sum(toint(properties.diskSizeGB)) by DiskSku=tostring(sku.name), location
```

## Count of Virtual Machine by Size and Location
```
Resources
| where type == "microsoft.compute/virtualmachines"
| summarize Count=count(properties.hardwareProfile.vmSize) by OS=tostring(properties.storageProfile.osDisk.osType), location, vmSize=tostring(properties.hardwareProfile.vmSize)
```

## Count of VMs by Size and Location
```
```
