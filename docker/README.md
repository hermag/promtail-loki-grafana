## Deployment of Promtail, Loki and Grafana

### docker-compose file

docker-compose file deployes 4 containers of Loki, 4 containers of Promtail and 1 container of Grafana dashboard. 

#### Promtail
The ```Promtail``` containers have two mounted volumes, ```/var/log``` and ```./promtail_0```. The ```/var/log``` folder
contains all the log files, which are parsed and pushed to ```Loki``` by the ```Promtail```. In the ```./promtail_0```
folder there is a configuration file for the first instance/service/container of promtail, which looks like this:
```
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki0:3100/loki/api/v1/push

scrape_configs:

# local machine logs

- job_name: local
  pipeline_stages:
  #whatever tags you can extract from your json logs
    - json:
        expressions:
          tenant_id: Tenant_0
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/Tenant_0/*log
```
Here the ```http_listen_port``` can be used to push the logs directly from the applications/services and in such way, one can deliver the application 
logs into the ```Loki``` and afterwards to ```Grafana```. In the ```clients``` section, the ```- url: http://loki0:3100/loki/api/v1/push``` is one of the most important
lines. It contains ```Loki``` API address. In this case, by specifying ```loki0:3100``` we refer to the service/container with name ```loki0```, since it 
is running in the same network as ```Promtail``` and ```Grafana``` services, it can be reached by the name of the service. The final most important line 
in ```Promtail``` configuration is ```__path__: /var/log/Tenant_0/*log```, this path points to the location of all log files, and since we have mounted the 
```/var/log``` volume of our host Linux/Mac OS, ```Promtail``` container/service will look for all log files on our docker host machine.

In the ```docker-compose``` file the ```Promtail``` service is configured as follows:

```
  promtail0:
    image: grafana/promtail:2.4.0
    volumes:
      - /var/log:/var/log
      - ./promtail_0:/etc/promtail
    # ports:
    #   - "1514:1514" # this is only needed if you are going to send syslogs
    restart: unless-stopped
    command: -config.file=/etc/promtail/promtail-config.yml
    networks:
          - loki
```
The key part here is the following: ```command: -config.file=/etc/promtail/promtail-config.yml```, where the configuration file is pointing to the one, which is located in the ```promtail_0``` folder on the host OS. 

#### Grafana Loki
Since we 


```docker
version: "3"
networks:
  loki:
services:
  #################
  loki0:
    image: grafana/loki:2.4.0
    volumes:
      - ./loki_0:/etc/loki
    ports:
      - "3100:3100"
    restart: unless-stopped
    command: -config.file=/etc/loki/loki-config.yml
    networks:
      - loki
```
