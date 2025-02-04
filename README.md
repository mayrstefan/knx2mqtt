# knx2mqtt - A KNX2MQTT Bridge allowing bidirectional telegram transfer

I've created this project as a replacement for the KNX integration of [HomeAssistant](https://home-assistant.io/) that worked not stable in my environment.

It is quite simple and does what it's name says: It works as a bridge between KNX and MQTT tranlating messages between these in both directions.

## Installation

The installation requires at least Python 3.7 and `git`.

I usually install my own services under `/opt/services`.
So, it works out of the box, if you just do:

```
mkdir -p /opt/services
cd /opt/services
git clone https://github.com/gbeine/knx2mqtt.git
cd knx2mqtt
./install
```

The `install` script creates a virtual python environment using the `venv` module.
All required libraries are installed automatically.

## Configuration

The configuration is located in `knx2mqtt.yaml`.
Place it under `/etc/knx2mqtt/knx2mqtt.yaml` or in the directory where you run knx2mqtt.
Do the same with the `logging.conf` file.

### MQTT

First, you need to configure your MQTT server.

```
mqtt:
    host: your.mqtt-server.name
    port: 1883
    user: knx2mqtt
    password: topsecret
    topic: "home/bus/knx"
    qos: 0
    retain: true
```

Usually, you only need to change the `host`, `user` and `password`.
Maybe, you want to use another `topic`.
Leave `qos` and `retain` unless you know what these parameters do.

### KNX

Currently, only a subset of the xknx configuration options is supported.
It may become more in the future, if I found testing environments with according setups.
The configuration is now in the Home Assistant style, so if you have an older `xknx.yaml`,  convert your configuration with [XKNX config converter](https://xknx.io/config-converter/).

```
knx:
  individual_address: 15.15.249
  tunneling:
    host: 192.168.0.11
    local_ip: 192.168.0.12
```

Currently, only tunneling is supported as configuration option.
Feel free to add routing or other options and open a pull request for this.

### Items

Then you can configure your bus topology as items.

```
items:
- address: 0/8/15
  type: DTPTemperature
- address: 4/7/15
  type: DTPHumidity
  ...
```

Each item need an `address` (the group address) and a `type`.
Unfortunately, the list of types is not part of the xknx documentation.
But the examples in the file I provide with the project may fit for the most purposes.
All supported types can be found in the [xknx sources](https://github.com/XKNX/xknx/blob/main/xknx/dpt/__init__.py).

The default operating mode for an object is to listen on the KNX and publish the telegram values to MQTT.

That may be changed using the following settings:

* `mqtt_subscribe` (default: false): if set to `true`, changes on any related MQTT topic will be processed
* `mqtt_publish` (default: true): if set to `true`, the values for this item will be published on all related MQTT topics 
* `knx_subscribe` (default: true): if set to `true`, changes on any related KNX address will be processed
* `knx_publish` (default: false): if set to `true`, the values for this item will be published on all related KXN addresses 

To prevent communication loops, the bridge caches all states that have been published.
If an event for an item is received, the bridge checks if the value has changed. 
The value will be published only to addresses with valued that differ from the current one.
This works, no matter if the source of the event is MQTT or KNX.

**Attention:** You can still configure loops using the same MQTT topic or KNX address for different things!

It is possible to add addtional MQTT topics and KNX addresses.
Examples are located in the configuration file in the project.

### Publishing

All values are published using the group address and the MQTT topic.

So, the Date exposing sensor in the example is listening for `home/bus/knx/0/0/1` and the switch is listening on and publishing to `home/bus/knx/0/1/1`.

## Running knx2mqtt

I use [Supervisor](http://supervisord.org/) to manage my local services.

For this, a configuration file and an executable are part of the project.

The configuration file is located under `supervisor`, just copy or link it to `/etc/supervisor/conf.d`.

The `run` script expects an environment variable named `LOGDIR` where the logfile should be written. This is set by the supervisor configuration, so change it there.

## Support

I have not the time (yet) to provide professional support for this project.
But feel free to submit issues and PRs, I'll check for it and honor your contributions.

## License

The whole project is licensed under MIT license. Stay fair.
