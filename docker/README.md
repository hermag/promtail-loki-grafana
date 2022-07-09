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

**Important Note**: Make sure to have .log text files in ```/var/log/``` folder. You can use custom file types as well as custom locations.

#### Grafana Loki

In this setup, Loki is used to accept the log data sent by the different agents, including Promtail. In the docker-compose.yaml file, it's section looks like this:

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
In this configuration, the most important line is this one ```- ./loki_0:/etc/loki``` and this ```command: -config.file=/etc/loki/loki-config.yml```. The first points to the folder, where the Loki configuration file is located, while the second one specifies the argument to use the provided configuration file.
Configuration file looks as follows:
```
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093

```
The key line here is this ```http_listen_port: 3100```. Here we are specifying the Loki service port, which will be listened. Since we are running multiple Loki services, the ports will be different, hence in promtail configuration, we should not forget to specify the correct port and the corresponding service name.
- promtail_0 -> loki_0 (loki0:3100)
- promtail_1 -> loki_1 (loki1:3101)
- promtail_2 -> loki_2 (loki2:3102)
- promtail_3 -> loki_3 (loki3:3103)


#### Grafana
Running grafana dashboard is trivial, for that we have the following section in the ```docker-compose.yml``` file.
```
  grafana:
    image: grafana/grafana:latest
    user: "1000"
    volumes:
    - ./grafana:/var/lib/grafana
    ports:
      - "3000:3000"
    restart: unless-stopped
    networks:
      - loki
```
The most important line here is ```./grafana:/var/lib/grafana``` this we need to keep the grafana changes persistant. The ports section is trivial, since Grafana is running on port 3000 and we would like to keep the same port in use (after launching the grafana service, we can access it on http://localhost:3000 , the default username is ```admin``` and password ```admin```, once the password changed it will stay changed since we keep the grafana configuration persistantly in ```./grafana``` folder.)

After deployment and logging in to Grafana we need to add the Organisations plugin, for that we need to go to the following settings:
```Configuration --> Plugins``` search for Organisations 