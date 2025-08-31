# Network Monitoring using Grafana, Prometheus and Ping ExporterMonitoring

Basic Monitoring using ICMP / Ping for network devices

## Instructions
These instructions accompany the [YouTube instructional video by House of Logic](https://youtu.be/fnoTHoZzNSc).

The Ping Exporter can be found [here.](https://github.com/czerwonk/ping_exporter)

Note that this repo now also contains details of using the Blackbox exporter for application level monitoring. The blackbox exporter can be used alongside the ping exporter (or any other exporter in fact), so you can add to your prometheus configuration rather than replacing it. Please see the blackbox subdirectory for details.

### Clone Repo

Clone this repo and edit the files in the ping_exporter and prometheus directories as appropriate with the correct host and port details.

### Install docker on a host of your choice

```
sudo apt-get update

sudo apt-get install docker.io
```

### Install ping exporter using docker (running as daemon)

Run the ping_exporter docker container, mapping the config.yml you have edited.

```
sudo docker run --init -d -p 9427:9427 -v /home/ubuntu/ping_exporter/config.yml:/config/config.yml --name ping_exporter czerwonk/ping_exporter

```

Check the container is running:

```
sudo docker ps

curl localhost:9427

```

(or open in Browser)

### Configure Prometheus using nano 

```
cd ../prometheus

nano prometheus.yml
```

## Start Prometheus container using Docker

```
cd ../prometheus
sudo docker volume create prometheus-data
sudo docker run --name prometheus \
    -p 9090:9090 \
    -d \
    -v /home/ubuntu/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
    -v prometheus-data:/prometheus \
    prom/prometheus
```

### Setup Grafana using docker

Create persistent volume for your data

```
sudo docker volume create grafana-storage

sudo docker run -d -p 3000:3000 --name=grafana \
  --volume grafana-storage:/var/lib/grafana \
  grafana/grafana-enterprise
```

Login to grafana using browser to connect to Grafana port 3000, eg http://192.168.2.11:3000

Default credentials

username = admin

password = admin 

### Create dashboard in grafana using UI

Custom label:
```
{{target}}
```

## Useful links

[Ping Exporter](https://github.com/czerwonk/ping_exporter)

[Installing Prometheus using Docker](https://prometheus.io/docs/prometheus/latest/installation/#using-docker)

[Installing Grafana using Docker](https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/#run-grafana-docker-image)

 
