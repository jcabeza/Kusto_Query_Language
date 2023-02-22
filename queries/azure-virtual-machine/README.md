# KQL Queries for Azure Virtual Machines

## List of Virtual Machines Time Creation
```
Resources 
| where type == "microsoft.compute/virtualmachines"
| extend props=parse_json(properties)
| project name, resourceGroup, time_created=props["timeCreated"]
```

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

## Count Total Disk Size (GB)
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

## List of Virtual Machine NICs and Public IPs
```
Resources
| where type =~ 'microsoft.compute/virtualmachines'
| extend nics=array_length(properties.networkProfile.networkInterfaces)
| mv-expand nic=properties.networkProfile.networkInterfaces
| where nics == 1 or nic.properties.primary =~ 'true' or isempty(nic)
| project vmId = id, vmName = name, vmSize=tostring(properties.hardwareProfile.vmSize), nicId = tostring(nic.id)
  | join kind=leftouter (
    Resources
    | where type =~ 'microsoft.network/networkinterfaces'
    | extend ipConfigsCount=array_length(properties.ipConfigurations)
    | mv-expand ipconfig=properties.ipConfigurations
    | where ipConfigsCount == 1 or ipconfig.properties.primary =~ 'true'
    | project nicId = id, privateIP= tostring(ipconfig.properties.privateIPAddress), publicIpId = tostring(ipconfig.properties.publicIPAddress.id), subscriptionId) on nicId
| project-away nicId1
| summarize by vmId, vmSize, nicId, privateIP, publicIpId, subscriptionId
  | join kind=leftouter (
    Resources
      | where type =~ 'microsoft.network/publicipaddresses'
        | project publicIpId = id, publicIpAddress = tostring(properties.ipAddress)) on publicIpId
| project-away publicIpId1
| sort by publicIpAddress desc
```

## List of Virtual Machine with Storage Profile
```
Resources
| where type contains "microsoft.compute/disks"
| project Os=properties.osType,DiskSku=sku.name,DiskSizeGB=properties.diskSizeGB,id = managedBy
| join (Resources 
| where type == "microsoft.compute/virtualmachines") on id
```
```
Resources
| where type == "microsoft.compute/virtualmachines"
| extend osDiskId= tostring(properties.storageProfile.osDisk.managedDisk.id)
    | join kind=leftouter(
        resources
            | where type =~ 'microsoft.compute/disks'
            | where properties !has 'Unattached'
            | where properties has 'osType'
            | project OS = tostring(properties.osType), osSku = tostring(sku.name), osDiskSizeGB = toint(properties.diskSizeGB), osDiskId=tostring(id)) on osDiskId
    | join kind=leftouter(
        resources
			| where type =~ 'microsoft.compute/disks'
            | where properties !has "osType"
            | where properties !has 'Unattached'
            | project sku = tostring(sku.name), diskSizeGB = toint(properties.diskSizeGB), id = managedBy
            | summarize sum(diskSizeGB), count(sku) by id, sku) on id
| project vmId=id, OS, location, resourceGroup, subscriptionId, osDiskId, osSku, osDiskSizeGB, DataDisksGB=sum_diskSizeGB, diskSkuCount=count_sku
| sort by diskSkuCount desc
```

## List of Virtual Machines Deallocated
```
Resources
| project name, location, PowerState=tostring(properties.extended.instanceView.powerState.code), type
| where type =~ 'Microsoft.Compute/virtualMachines'
| where PowerState == "PowerState/deallocated"
| order by name desc
```