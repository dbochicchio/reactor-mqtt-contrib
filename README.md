# reactor-mqtt-contrib
Additional MQTT templates for Reactor.

# Installation
Download all the yaml files and save them under *reactor/config/mqtt_templates*. If *mqtt_templates* directory does not exist, simply create it.
Copy the files and restart Reactor. Every time you update the files, a restart is needed.

# Configuration Example

Let's say you want to map a Shelly HEM. In your *reactor.yaml*, under *controllers:*, search for *mqtt*, then *config* and add:
```
  - id: mqtt
    name: MQTT
    ...
    config:
      ...
      entities:
        # energy
        shelly_hem:
          name: "Solar"
          topic: "shelly-solar"
          uses_template: shelly_hem
          channel: 0    
```

Where *shelly-solar* is the device name (configured in your Shelly as the MQTT topic) and channel is the index (Shelly HEM supports 2 clamps).
Look at the table below for other parameters.

All the templates are supporting query/init, and at startup their state will be updated. *x_mqtt.poll* could be used to poll specific devices in reaction.

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
| switchbot_switch | Switchbot Switch mapped from https://github.com/fphammerle/switchbot-mqtt | power_switch, toggle, battery_power | topic |