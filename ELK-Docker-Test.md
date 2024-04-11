# Run ElasticSearch Stack with Docker

## Configure Ubuntu

Update apt packages
```
sudo apt update
```
If you want to connect using SSH remotely, you might want to install ssh
```
sudo apt install ssh
```
If using Hyper-V you might want to use Azure kernel (works on Hyper-V)
```
sudo apt install linux-azure
```
Disable unattended-upgrades, select No after running the following command
```
sudo dpkg-reconfigure unattended-upgrades
```
Uninstall old versions of docker
```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
Set up Docker's apt repository.
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install the latest version of docker
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


Manage Docker as a non-root user, add your user to the docker group.
```
sudo usermod -aG docker $USER
```
Then logout and login or use the following command
```
su - ${USER}
```
Test your groups' membership
```
groups
```
Verify that the Docker Engine installation is successful by running the hello-world image.
```
sudo docker run hello-world
```
If you have images stored on another machine, you can export them from there and import them in our current machine
```
docker save -o elastic.tar <image name>
docker save -o kibana.tar <image name>
docker save -o heartbeat.tar <image name>
```
Copy the tar files to the new machine
```
scp
```
And then import images
```
docker load -i elastic.tar
docker load -i kibana.tar
docker load -i heartbeat.tar
```
Test if images are imported properly
```
docker images
```
If you want to download images directly from docker hub
```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.13.0-amd64
docker pull docker.elastic.co/kibana/kibana:8.13.0-amd64
docker pull docker.elastic.co/beats/heartbeat:8.13.0-amd64
```
To view system-wide information about Docker
```
docker info
```
Create a new docker network.
```
docker network create elastic
```

Create a new docker network.
```
docker network create elastic
```

Get the ElasticSearch image
```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.13.0-amd64
```

Start an Elasticsearch container.
```
docker run --name es01 --net elastic -p 9200:9200 -it -m 1GB docker.elastic.co/elasticsearch/elasticsearch:8.13.0-amd64
```

Copy the generated elastic password and enrollment token. These credentials are only shown when you start Elasticsearch for the first time. You can regenerate the credentials using the following commands.
```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

Copy the http_ca.crt SSL certificate from the container to your local machine.
```
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .
```

Make a REST API call to Elasticsearch to ensure the Elasticsearch container is running.
```
curl --cacert http_ca.crt -u elastic:$ELASTIC_PASSWORD https://localhost:9200
```

Get Elastic IP Address
```
docker inspect es01
```

## Run Kibana

Get Kibana image.
```
docker pull docker.elastic.co/kibana/kibana:8.13.0-amd64
```

Start a Kibana container.
```
docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.13.0-amd64
```

Get Kibana IP Address
```
docker inspect kib01
```

## Run Heartbeat

Pull the Heartbeat Docker image.
```
docker pull docker.elastic.co/beats/heartbeat:8.13.0-amd64
```

If elastic and kibana container not running, you can start them
```
docker start es01
docker start kib01
```

to use names instead of IP addresses in configuration we can make changes to the hosts file
```
sudo nano /etc/hosts
```

Then use the IP addresses you got from inspecting the container configuration for each of elastic and kibana
```
172.18.0.2      elastic
172.18.0.3      kibana
```
Create the configuration file for heartbeat
```
nano heartbeat.yml
```

Fill the file "heartbeat.yml" with the following content
```
heartbeat.monitors:
  - type: "http"
    schedule: '@every 5s'
    urls:
      - "http://elastic:9200"
      - "http://kibana:5601"
  - type: "icmp"
    schedule: '@every 5s'
    hosts:
      - "elastic"
      - "kibana"

output.elasticsearch:
  hosts: ["172.28.0.2:9200"]  # We are using the IP address instead of the DNS name because the certificate we are using is self-signed and it will not work properly
  username: "elastic"
  password: "x7vR1V=IgFfa_5fcB+va"
  protocol: "https"
  ssl.certificate_authorities: ["/usr/share/heartbeat/http_ca.crt"]

setup.kibana:
  host: "http://kibana:5601"
```

Run heartbeat container with this command
```
docker run -d \
  --name=heartbeat \
  --user=heartbeat \
  --network=elastic \
  --volume="$(pwd)/heartbeat.yml:/usr/share/heartbeat/heartbeat.yml:ro" \
  --volume="$(pwd)/http_ca.crt:/usr/share/heartbeat/http_ca.crt:ro" \
  --cap-add=NET_RAW \
  docker.elastic.co/beats/heartbeat:8.13.0-amd64 \
  --strict.perms=false -e
```

Enter the shell of the heartbeat container
```

```

Test the config of heartbeat
```

```

Setup heartbeat
```

```

Now login to kibana using http://178.28.0.3:5601 and enter username and password

Go to Uptime and check the heartbeat of the configured monitors








https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

Sources
https://docs.docker.com/engine/install/ubuntu/
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04
https://computingforgeeks.com/how-to-export-and-import-docker-images-containers/








