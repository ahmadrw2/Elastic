# ELK Task Test

We will install ElasticSearch and Kibana into a Linux VM hosted on Hyper-V

## Table of Contents
### 1. Setup & Configure Ubuntu VM
### 2. Install ElasticSearch
### 3. Install Kibana
### 4. Install & Configure Nginx
### 5. Create Role & Username
### 6. Install metricbeat
### 7. Update the "safee" role to access metric data as read only
### 8. Install filebeat
### 9. Update the "safee" role to access logs data as read only
### 10. Install heartbeat

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

### 1.5 Configure Ubuntu
From main menu, select File, then Settings, then Network Adapter, then select the Hyper-V Virtual Switch.

Right click on the desktop and select "Open Terminal"

Disable unattended upgrades, and select No
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

## 2. Install ElasticSearch

Get the IP address of your VM. In our case it was 192.168.1.5
```
ifconfig
```
On your host, copy ElasticSearch deb package from your computer. Make sure that the files are in the current directory in Windows. Open a new terminal windows and enter the command
```
scp "elasticsearch-8.12.2-amd64.deb" ahmad@192.168.1.5:~/elasticsearch-8.12.2-amd64.deb
```
Enter "Yes" when prompted and then enter your Ubuntu password to continue.

Start SSH session from your host to your VM.
```
ssh ahmad@192.168.1.5
```
Install ElasticSearch
```
sudo dpkg -i elasticsearch-8.12.2-amd64.deb
```
Make sure that you copy your super user password for elastic
> The generated password for the elastic built-in superuser is : a23LUAp*Cn3GEGg3zngN

Enable Elasticsearch to run as a service
```
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
```
Configure Elasticsearch node for connectivity
```
sudo nano /etc/elasticsearch/elasticsearch.yml
```
And change the following settings
```
cluster.name: elasticsearch-demo
network.host: 192.168.1.5
transport.host: 0.0.0.0
```
Press Ctrl+O to save the file and press Enter, and then Ctrl+X to exit

start the Elasticsearch service
```
sudo systemctl start elasticsearch.service
```
Make sure that Elasticsearch is running properly.
```
sudo curl --cacert /etc/elasticsearch/certs/http_ca.crt -u elastic:a23LUAp*Cn3GEGg3zngN https://localhost:9200
```
You must get a response such as
> {
  "name" : "ELK-VM-Test-04",
  "cluster_name" : "elacticsearch-demo",
  "cluster_uuid" : "fHiVnCzVQ-qMksMXZ9IpBQ",
  "version" : {
    "number" : "8.12.2",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "48a287ab9497e852de30327444b0809e55d46466",
    "build_date" : "2024-02-19T10:04:32.774273190Z",
    "build_snapshot" : false,
    "lucene_version" : "9.9.2",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}

Check the status of Elasticsearch:
```
sudo systemctl status elasticsearch
```

## 3. Install Kibana

On your host, copy ElasticSearch deb package from your computer. Make sure that the files are in the current directory in Windows. Open a new terminal windows and enter the command
```
scp "kibana-8.12.2-amd64.deb" ahmad@192.168.1.5:~/kibana-8.12.2-amd64.deb
```
Install Kibana
```
sudo dpkg -i kibana-8.12.2-amd64.deb
```
Generate a Kibana enrollment token
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana

Copy the generated enrollment token from the command output
> eyJ2ZXIiOiI4LjEyLjIiLCJhZHIiOlsiMTkyLjE2OC4xLjU6OTIwMCJdLCJmZ3IiOiJkMDRlNDg2NGMzYzg5ZDNlMmQwMDY5Yjc3YWY2M2QxODkzOTFhM2ZmY2I0ZmIzMzkyMTdmYzM3OGE3MmM0MDk2Iiwia2V5IjoiVmtWRWRZNEJwOWlxeW9Ec29RaFk6X2ItV1FIQXVSQ2VPVkI0ekRYV1dTQSJ9

enable Kibana to run as a service
```
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
```
Open the Kibana configuration file for editing
```
sudo nano /etc/kibana/kibana.yml
```
and make sure that the following config is set to localhost
```
elasticsearch.hosts: ["https://localhost:9200"]
```
Start the Kibana service
```
sudo systemctl start kibana.service
```
Get details about the Kibana service
```
sudo systemctl status kibana
```
A URL is shown, copy it with the key to get started
http://localhost:5601/?code=579443

When presented, enter the previous enrollment token

Note: if you are using ssh to configure your VM and you need to copy the enrollment token to your VM, you can paste it into a text file and then copy it to your VM
```
scp "Desktop\token.txt" ahmad@192.168.1.5:~/token.txt
```
And then click on "Configure" and wait to Kibana to finish setting up

When you see the Welcome to Elastic page, provide the elastic as the username and provide the elastic password
>a23LUAp*Cn3GEGg3zngN

## 4. Install & Configure Nginx

Install nginx
```
sudo apt install nginx
```
Check status of nginx
```
systemctl status nginx
```
Configure your server
```
sudo nano /etc/nginx/sites-available/kibana
```
Configure as follows

>server {
    listen 8888;
    listen [::]:8888;

    server_name kibana.site;
        
    location / {
        proxy_pass http://localhost:5601;
        include proxy_params;
    }
}

Enter Ctrl+O to save and then enter, then enter Ctrl+X to close

Enable this configuration file by creating a link from it to the sites-enabled directory that Nginx reads at startup:
```
sudo ln -s /etc/nginx/sites-available/kibana /etc/nginx/sites-enabled/
```
You can now test your configuration file for syntax errors:
```
sudo nginx -t
```
With no problems reported, restart Nginx to apply your changes:
```
sudo systemctl restart nginx
```
Now you can access Kibana from remote hosts using the nginx reverse proxy URL which is http://192.168.1.5:8888

## 5. Create Role & Username

- Login to Kibana
- Stack Management
- Security
- Roles
- Click on Create Role
- Create a new role named "safee" 
- Stack Management
- Security
- Users
- Click on Create User
- enter the name as "test" with the other details. In Privileges, select "safee" as the role.

## 6. Install metricbeat

Copy the installation package
```
scp "Downloads\metricbeat-8.12.2-amd64.deb" ahmad@192.168.1.5:~/metricbeat-8.12.2-amd64.deb
```
Install metricbeat
```
sudo dpkg -i metricbeat-8.12.2-amd64.deb
```
Check metricbeat modules
```
sudo metricbeat modules list
```
Test metricbeat
```
sudo metricbeat test config
```
Test output
```
sudo metricbeat test output
```
We got an erorr accessing elastic on http://localhost:9200

To fix the error, we need to go to metericbeat configuration and add the following configuration
```
sudo nano /etc/metricbeat/metricbeat.yml
```
Edit the configuration file as follows
```
protocol: "https"
username: "elastic"
password: "a23LUAp*Cn3GEGg3zngN"
ssl_certificate_authorities: ["/etc/elasticsearch/certs/http_ca.crt"]
```
Restart metricbeat service
```
sudo systemctl restart metricbeat.service
```
Test again to see if all is working
```
sudo metricbeat test output
```
Setup metricbeat and add it dashboard to Kibana
```
sudo metricbeat setup
```
Enable metericbeat service
```
sudo systemctl enable metricbeat.service
```
## 7. Update the "safee" role to access metric data as read only

- Open Kibana home page
- Stack Management
- Security
- Roles
- Edit "safee" role
- Add Kibana privileges
- In spaces select "All Spaces"
- In Privileges for all features, select "Customize"
- In Observability, Metrics, select "read"
- Click on "Create Global Privileges"
- Click on "Update Role"

## 8. Install filebeat

Copy the installation package
```
scp "Downloads\filebeat-8.12.2-amd64.deb" ahmad@192.168.201.5:~/filebeat-8.12.2-amd64.deb
```
Install filebeat
```
sudo dpkg -i filebeat-8.12.2-amd64.deb
```
Check filebeat modules
```
sudo filebeat modules list
```
Enable required modules
```
sudo filebeat modules enable system
sudo filebeat modules enable auditd
```
Test filebeat
```
sudo filebeat test config
```
Test output
```
sudo filebeat test output
```
We got an erorr accessing elastic on http://localhost:9200

To fix the error, we need to go to filebeat configuration and add the following configuration
```
sudo nano /etc/filebeat/filebeat.yml
```
Edit the configuration file as follows
```
protocol: "https"
username: "elastic"
password: "a23LUAp*Cn3GEGg3zngN"
ssl.certificate_authorities: ["/etc/elasticsearch/certs/http_ca.crt"]
```
Restart filebeat service
```
sudo systemctl restart filebeat.service
```
Test again to see if all is working
```
sudo filebeat test output
```
Setup filebeat and add it dashboard to Kibana
```
sudo filebeat setup
```
Enable filebeat service
```
sudo systemctl enable filebeat.service
```

## 9. Update the "safee" role to access log data as read only

- Open Kibana home page
- Stack Management
- Security
- Roles
- Edit "safee" role
- Update Custom Privileges
- In Observability, Logs, select "read"
- Click on "Update Global Privileges"
- Click on "Update Role"

## 10. Install heartbeat

Copy the installation package
```
scp "Downloads\heartbeat-8.12.2-amd64.deb" ahmad@192.168.201.5:~/heartbeat-8.12.2-amd64.deb
```
Install heartbeat
```
sudo dpkg -i heartbeat-8.12.2-amd64.deb
```
Go to heartbeat configuration and add the following configuration
```
sudo nano /etc/heartbeat/heartbeat.yml
```
Edit the configuration file as follows
```
protocol: "https"
username: "elastic"
password: "a23LUAp*Cn3GEGg3zngN"
ssl.certificate_authorities: ["/etc/elasticsearch/certs/http_ca.crt"]
```
Configure heartbeat monitors
```
sudo nano /etc/heartbeat/heartbeat.yml
```
Enter the following configuration
```
heartbeat.monitors:
- type: http
  enabled: true
  id: my-monitor
  name: My Monitor
  urls: ["https://localhost:9200", "http://localhost:5601"]
  schedule: '@every 10s'
```
Save the configuration Ctrl+O and press Enter, then Ctrl+X to exit

Test heartbeat config
```
sudo heartbeat test config
```
Setup heartbeat
```
sudo heartbeat setup
```
Start Heartbeat
```
sudo systemctl start heartbeat-elastic
sudo systemctl enable heartbeat-elastic
```
Go to the following URL
```
http://192.168.101.166:8888/app/uptime
```
Done


### Sources
- https://www.elastic.co/guide/en/elastic-stack/8.12/installing-stack-demo-self.html#install-stack-self-elasticsearch-first
- https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-on-ubuntu-22-04
- https://www.elastic.co/guide/en/beats/heartbeat/current/heartbeat-installation-configuration.html
