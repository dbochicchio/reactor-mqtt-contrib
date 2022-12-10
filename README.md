# reactor-mqtt-contrib
Additional MQTT templates for Reactor.

# Installation
Download all the yaml files and save them under *reactor/config/mqtt_templates*. If *mqtt_templates* directory does not exist, simply create it.
Copy the files and restart Reactor. Every time you update the files, a restart is needed.

# Templates

| Template ID | Device | Capabilities | Parameters |
| ------------- | ------------- | ------------- | ------------- |
| shelly_hem | Shelly Home Energy Meter | power_sensor, energy_sensor, voltage_sensor, current_sensor, power_factor_sensor, x_energy_sensor_exported | topic, channel |
| shelly_relay_power | Shelly relay with power meter | switch, toggle, power_sensor, energy_sensor, voltage_sensor, current_sensor, power_factor_sensor | topic, channel |
| shelly_binary | Shelly with detached inputs, mapped as binary sensor | binary_sensor | topic, channel |
| shelly_exttemperature | Shelly with detached inputs, mapped as binary sensor | binary_sensor | topic, channel |
| shelly_exthumidity | Shelly with external humidity sensor | humidity_sensor | topic, channel |
| tasmota_sensor_pressure | Tasmota with Pressure sensor | value_sensor | topic, source |
| tasmota_sensor_illuminance | Tasmota with Illuminance sensor | light_sensor | topic, source |
| switchbot_switch | Switchbot Switch mapped from (switchbot-mqtt|https://github.com/fphammerle/switchbot-mqtt) | power_switch, toggle, battery_power | topic |
| owntracks_sensor | OwnTracks Sensor with multiple informations (position, current region, device battery). See additional configuration for info. | string_sensor, binary_sensor, battery_power, battery_maintenance, location | prefix, topic,  homeRegionName |

All the templates are supporting query/init, and at startup their state will be updated. *x_mqtt.poll* could be used to poll specific devices in reaction.

# Configuration

Let's say you want to map a Shelly HEM. In your *reactor.yaml*, under *controllers:*, search for *mqtt*, then *config* and add:
```
  - id: mqtt
    name: MQTT
    ...
    config:
      ...
      entities:
        # solar
        shelly_hem:
          name: "Solar"
          uses_template: shelly_hem
          topic: "shelly-solar"
          channel: 0    
```

Where *shelly-solar* is the device name (configured in your Shelly as the MQTT topic) and channel is the index (Shelly HEM supports 2 clamps).
Look at the table above for other parameters.

## Additional configuration

### OwnTracks

This template will create a device that has:
 - a string sensor with the current region
 - a binary sensor that's true when the user is at home
 - locations info via *location* capability
 - battery info for the device (including percentage and charging status)

Please refer to (OwnTracks documentation|https://owntracks.org/booklet/guide/topics/) for more info about the topics structure.

Let's suppose your base topic for owntracks is `owntracks/daniele/iPhone`.

In your *reactor.yaml*, under *controllers:*, search for *mqtt*, and then under *config* add this configuration:

```
        ...
        owntracks_daniele: #entity ID - must be unique
          name: "Daniele's Location" # friendly name
          prefix: "daniele"
          topic: "iPhone"
          uses_template: owntracks_sensor
          homeRegionName: "home"
```

Where `prefix` is the first part of the MQTT topic (*daniele* in our case) and `topic` is the last part, typically the device (*iPhone*).
`homeRegionName` is the name of your home region and will be used in order to update the binary sensor. It's case insensitive.

If you have multiple devices to track, just repeat the same configuration, using a different entity ID.