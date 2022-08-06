# mqtt-telegraf-influxdb-grafana-nodered-wireguard
Docker compose repo with MQTT, Telegraf, InfluxDB, Grafana, Node-RED, WireGuard

1 step:

mkdir -m 777 -p ~/mosquitto/config && \
mkdir -m 777 -p ~/mosquitto/data && \
mkdir -m 777 -p ~/mosquitto/log && \
mkdir -m 777 -p ~/influxdb/data && \
mkdir -m 777 -p ~/influxdb/conf && \
mkdir -m 777 -p ~/telegraf/conf && \
mkdir -m 777 -p ~/grafana/data && \
mkdir -m 777 -p ~/grafana/conf && \
mkdir -m 777 -p ~/grafana/log && \
mkdir -m 777 -p ~/node-red/data && \
mkdir -m 777 -p ~/wireguard/config

2 step:

cat > ~/mosquitto/config/mosquitto.conf <<EOF
listener 1883
#allow_anonymous false
#password_file /mosquitto/config/password.txt
EOF

3 step:

cat > ~/grafana/conf/grafana.ini <<EOF
[server]
http_port = 80
EOF

4 step:

cat >  ~/telegraf/conf/telegraf.conf <<EOF
[agent]
  interval = "3s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "3s"
  flush_jitter = "0s"
  precision = ""
  hostname = ""
  omit_hostname = false

[[outputs.influxdb_v2]]
  urls = ["http://192.168.50.56:8086"]
  token = "a"
  organization = "IoT"
  bucket = "IoT"

[[inputs.mqtt_consumer]]
  servers = ["tcp://192.168.50.56:1883"]
  topics = ["#"]
  username = "IoT"
  password = "student"
  data_format = "value"
  data_type = "float"
EOF

5 step 

cat > docker-compose.yml <<EOF
version: "2"
services:
  influxdb:
    container_name: influxdb
    image: influxdb
    environment:
      - TZ=Europe/Moscow
    ports:
      - "8086:8086"
    volumes:
      - ~/influxdb/data:/var/lib/influxdb
    networks:
      - influxdb-net
    restart: always

  wireguard:
    container_name: wireguard
    image: lscr.io/linuxserver/wireguard:latest
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Moscow
      - SERVERURL=ru-dev-serv.asuscomm.com #optional
      - SERVERPORT=1871 #optional
      - PEERS=5 #optional
      - PEERDNS=1.1.1.1 #optional
      - INTERNAL_SUBNET=10.13.13.0 #optional
      - ALLOWEDIPS=0.0.0.0/0 #optional
      - LOG_CONFS=false #optional
    volumes:
      - ~/wireguard/config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: always

  telegraf:
    container_name: telegraf
    image: telegraf
    environment:
      - TZ=Europe/Moscow
    volumes:
      - ~/telegraf/conf/telegraf.conf:/etc/telegraf/telegraf.conf
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - telegraf-net
    restart: always
    
  grafana:
    container_name: grafana
    image: grafana/grafana
    environment:
      - TZ=Europe/Moscow
    ports:
      - "80:80"
    volumes:
      - ~/grafana/data:/var/lib/grafana
      - ~/grafana/log:/var/log/grafana
      - ~/grafana/conf/grafana.ini:/etc/grafana/grafana.ini
    links:
      - influxdb
    networks:
      - grafana-net
    restart: always

  mosquitto:
    container_name: mosquitto
    image: eclipse-mosquitto:latest
    environment:
      - TZ=Europe/Moscow
    volumes:
      - ~/mosquitto/:/mosquitto  
    ports:
      - 1883:1883
    networks:
      - mosquitto-net
    restart: always

  node-red:
    container_name: node-red
    image: nodered/node-red:latest
    environment:
      - TZ=Europe/Moscow
    ports:
      - "1880:1880"
    volumes:
      - ~/node-red/data:/data
    networks:
      - node-red-net
    restart: always

networks:
  node-red-net:
  influxdb-net:
  mosquitto-net:
  grafana-net:
  telegraf-net:
EOF

6 step

docker compose up -d

7 step

docker exec -it mosquitto mosquitto_passwd -c /mosquitto/config/password.txt IoT

8 step

cat > ~/mosquitto/config/mosquitto.conf <<EOF
listener 1883
allow_anonymous false
password_file /mosquitto/config/password.txt
EOF

9 step

docker restart mosquitto

10 step

docker exec -it node-red node-red-admin hash-pw

11 step (add hast to node config)

nano +76,5 ~/node-red/data/settings.js

12 step

docker restart node-red

13 step

copy influx token 

14 step: edit influx token in telegraf.conf

nano +15,12 ~/telegraf/conf/telegraf.conf

15 step

docker restart telegraf

16 step (wireguard peer conf recive)

docker exec -it wireguard /app/show-peer 1
