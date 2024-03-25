# ELK Task Test

We will install ElasticSearch and Kibana into a Linux VM hosted on Hyper-V

## Table of Contents
### 1. Setup Ubuntu VM
### 2. Configure Ubuntu
### 3. Install ElasticSearch
### 4. Install Kibana
### 5. Install Nginx
### 6. Create Role & Username

## 1. Setup Ubuntu VM

### 1.1 Create a New VM
Open a terminal window with admin rights
```
New-VM -Name ELK-VM-Test-04 -MemoryStartupBytes 4096MB -Path "C:\VMs" -NewVHDPath "C:\VHDs\ELK-VHD-Test-04.vhdx" -NewVHDSizeBytes 127GB -Generation 2
Set-VM -Name ELK-VM-Test-04 -ProcessorCount 4 -AutomaticCheckpointsEnabled 0 -StaticMemory
```
### 1.2 Configure the VM
Using Hyper-V Manager, Select the new VM, and from the action pane, click on settings
- Change secure boot in security settings and select the template as "Microsoft UEFI Certificate Authority"
- In SCSI Controller, add a DVD drive and mount the Ubuntu ISO to it
- In Integration Services, select Guest Services

### 1.3 Start the VM
```
Start-VM -Name ELK-VM-Test-04
```

### 1.4 Install Ubuntu
- Use default settings when installing Ubuntu
- Configure hostname, username and password
- Reboot to Ubuntu and log onto your user account

## 2 Configure Ubuntu

### 2.1 Connnect internet to the VM
From main menu, select File, then Settings, then Network Adapter, then select the Hyper-V Virtual Switch.

Right click on the desktop and select "Open Terminal"

Disable unattended upgrades, and select No for the first command
```
sudo dpkg-reconfigure unattended-upgrades
```

Update APT
```
sudo apt update
```

Install the required curl
```
sudo apt install curl
```

Install the required ssh to be able to connect remotely
```
sudo apt install ssh
```

Install Azure kernel to improve speed in Hyper-V
```
sudo apt install linux-azure
```

If you are using Ubuntu Server, you might want to remove snapd and cloud-init
```
sudo apt purge cloud-init -y
sudo apt remove snapd
```

Install net-tools to use ifconfig
```
sudo apt install net-tools
```

## 3. Install ElasticSearch

Copy ElasticSearch deb package from your computer

























