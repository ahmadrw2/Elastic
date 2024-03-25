# ELK Task Test

We will install ElasticSearch and Kibana into a Linux VM hosted on Hyper-V

## Table of Contents
### 1. Setup Ubuntu VM
### 2. Install ElasticSearch
### 3. Install Kibana
### 4. Install Nginx
### 5. Create Role & Username

## 1. Setup Ubuntu VM
Open a terminal window with admin rights
```
New-VM -Name ELK-VM-Test-04 -MemoryStartupBytes 4096MB -Path "C:\VMs" -NewVHDPath "C:\VHDs\ELK-VHD-Test-04.vhdx" -NewVHDSizeBytes 127GB -Generation 2
Set-VM -Name ELK-VM-Test-04 -ProcessorCount 4 -AutomaticCheckpointsEnabled 0 -StaticMemory
```

Using Hyper-V Manager, Select the new VM, and from the action pane, click on settings
Change secure boot in security settings and select the template as "Microsoft UEFI Certificate Authority"
In SCSI Controller, add a DVD drive and mount the Ubuntu ISO to it
In Integration Services, select Guest Services

Start the VM
```
Start-VM -Name ELK-VM-Test-04
```

Install Ubuntu using default settings

