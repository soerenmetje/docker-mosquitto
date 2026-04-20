# Docker Mosquitto

This repo contains an [Eclipse Mosquitto](https://mosquitto.org/) Docker compose setup. 
It uses the official [eclipse-mosquitto](https://hub.docker.com/_/eclipse-mosquitto/) Docker image.

I created this setup, to run an MQTT broker on a BananaPi M2 Berry in my local network.

## Preparation

Few manual steps are needed to set up some configuration.

### Create Mosquitto Config

1. Create the config file in `./config/mosquitto.conf`:

```shell
nano ./config/mosquitto.conf
```

```
# Config file for mosquitto

# Persistence disabled since not needed the my use case
persistence false
persistence_location /mosquitto/data/    

log_dest stdout

password_file /mosquitto/config/password.txt
#allow_anonymous true

# MQTTS listener
listener 1883 0.0.0.0
protocol mqtt

# WS Listener
#listener 8080 0.0.0.0
#protocol websockets
```

> For more options, see https://mosquitto.org/man/mosquitto-conf-5.html

2. Create an empty password file in `.config/password.txt`:

```shell
touch .config/password.txt
```

3. Update file owner to `mosquitto` (used inside the container to serve mqtt):

```shell
docker compose run --rm mosquitto chown -R mosquitto:mosquitto /mosquitto/config/
```

### Add Mosquitto Users

To add users to authenticate with the MQTT broker, 
replace `my-username` with your username and run the following command:

```shell
docker compose run --rm --user mosquitto mosquitto mosquitto_passwd -c /mosquitto/config/password.txt my-username
```
This will ask for a password. In the end, users will be stored in `.config/password.txt`.

## Getting Started

Start the Mosquitto broker:
```shell
docker compose up
```

## Bonus: Subscribe to a Topic
On your laptop, you may want to subscribe to a topic, e.g., for debugging.

### Approach 1: MQTTX
[MQTTX](https://mqttx.app/) provides a nice Docker image [emqx/mqttx-cli](https://hub.docker.com/r/emqx/mqttx-cli) for running an MQTT CLI client:

```shell
docker run -it --rm emqx/mqttx-cli
```
In the container, you can subscribe to a topic via:
```shell
mqttx sub -t 'my/topic/#' -h 192.168.178.39 -p 1883 -u my-username -P my-password
```

### Approach 2: Eclipse Mosquitto
Similarly, this is also possible using the same Eclipse Mosquitto container image as used as a broker before:

```shell
docker run -it --rm eclipse-mosquitto sh
```

In the container, you can subscribe to a topic via:
```shell
mosquitto_sub -t 'my/topic/#' -h 192.168.178.39 -p 1883 -u my-username -P my-password -d
```

> `-d` enables debug messages / more verbose output.


## Sources

- https://hub.docker.com/_/eclipse-mosquitto/
- https://mosquitto.org/documentation/