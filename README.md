# reactor-mqtt-contrib

This repo contains additional MQTT templates for Reactor.

# Installation

Download all the yaml files and save them under *reactor/config/mqtt_templates*. If *mqtt_templates* directory does not exist, simply create it.
Copy the files and restart Reactor. Every time you update the files, a restart is needed.
You could also just download all this repo, and copy the ZIP file in the same directory, instead of single files.

# How to update

Just replace the files using the same mechanism introduced for installation and restart Reactor.

# Templates

| Template ID | Device | Capabilities | Parameters |
| ------------- | ------------- | ------------- | ------------- |
| shelly_hem | Shelly Home Energy Meter | power_sensor, energy_sensor, voltage_sensor, current_sensor, power_factor_sensor, x_energy_sensor_exported | topic, channel |
| shelly_relay_simple | Shelly relay (simple version) | switch, toggle | topic, channel |
| shelly_relay_power | Shelly relay with power meter | switch, toggle, power_sensor, energy_sensor, voltage_sensor, current_sensor, power_factor_sensor | topic, channel |
| shelly_binary | Shelly with detached inputs, mapped as binary sensor | binary_sensor | topic, channel |
| shelly_exttemperature | Shelly with detached inputs, mapped as binary sensor | binary_sensor | topic, channel |
| shelly_exthumidity | Shelly with external humidity sensor | humidity_sensor | topic, channel |
| shelly_scenecontroller | Shelly as scene controller | button, scene_activation | topic, channel |
| shelly_button1 | Shelly Button 1 | button, battery_power, battery_maintenance | topic, channel |
| tasmota_sensor_pressure | Tasmota with Pressure sensor | value_sensor | topic, source |
| tasmota_sensor_illuminance | Tasmota with Illuminance sensor | light_sensor | topic, source |
| switchbot_switch | Switchbot Switch mapped from [switchbot-mqtt](https://github.com/fphammerle/switchbot-mqtt) | power_switch, toggle, battery_power | topic |
| fullykiosk | [Fully Kiosk](https://www.fully-kiosk.com/). See additional configuration for info. | string_sensor, binary_sensor, battery_power, battery_maintenance, dimming | topic |
| owntracks_sensor | OwnTracks Sensor with multiple informations (position, current region, device battery). See additional configuration for info. | string_sensor, binary_sensor, battery_power, battery_maintenance, location | prefix, topic, homeRegionName, notHomeRegionName |
| prism_solar_charger,  prism_solar_session | Prism Solar EV Charger from [Silla Industries](https://silla.industries/en/docs/prism/prism-use-and-maintenance/). See additional configuration for info. | ev_charger, power_switch, toggle, power_sensor, energy_sensor, voltage_sensor, current_sensor | topic, channel |

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

### Dual switch devices

Dual switch devices, such as Shelly 2 or 2.5, should be mapped using separate device for each channel. Simply specify *channel* parameters accordingly.

### Scene controllers

Many devices are also offering scene controller capabilities. Map a new device using *shelly_scenecontroller* as a template. Simply specify *channel* parameters accordingly for multi-buttons devices (Shelly 2.5, i3, etc).

### Shelly Uni

Shelly Uni is offering two inputs and two outputs, so you should map it using *shelly_relay_simple* (using *channel* as 0 or 1) and *shelly_binary* (using *channel* as 0 or 1).
You'll end up with 4 devices, 2 mapping the inputs and 2 mapping the outputs. If you have temperatures/humidity sensors, map a new device using *shelly_exttemperature* or *shelly_exthumidity*, setting the appropriate channels. You'll probably have 4-5 devices mapped, which gives you all the control over what's represented.

### OwnTracks

This template will create a device that has:
 - a string sensor with the current region
 - a binary sensor that's true when the user is at home
 - locations info via *location* capability
 - battery info for the device (including percentage and charging status)

Please refer to [OwnTracks documentation](https://owntracks.org/booklet/guide/topics/) for more info about the topics structure.

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
          notHomeRegionName: "not_home"
```

Where `prefix` is the first part of the MQTT topic (*daniele* in our case) and `topic` is the last part, typically the device (*iPhone*).
`homeRegionName` is the name of your home region and will be used in order to update the binary sensor. It's case insensitive.
`notHomeRegionName` is the name of the region when not home. If omitted, it's null.

If you have multiple devices to track, just repeat the same configuration, using a different entity ID.

### Prism Solar

To fully support this EV Charger, you'll need to map two devices:

```
        ...
        # prism
        prism_solar_charger:
          name: "Prism"
          topic: "prism"
          channel: 1
          uses_template: prism_solar_charger

        prism_solar_session:
          name: "Prism: session info"
          topic: "prism"
          channel: 1
          uses_template: prism_solar_session
```

The first device will implement the charger and its commands. The second one will show the information (power, current and energy) for the current charging session.
*channel* is usually 1, unless you have a Prism Solar Duo. In this case, you'll need to map multiple devices using *prism_solar_charger* as base template.
Neither poll nor LWT are supported by Prism Solar.
Even if night mode is supported via MQTT messages, it's better to create something with Reactor capabilities, because it has a cumbersome structure and it's mostly cloud-driven.

### Fully Kiosk

The device is mapping binary sensor (for screen turned on/off), dimming (for screen brigthness), battery level and charging status (plugged-in, battery power), and a *string_sensor* capability that's showing the current app. Unfortunately, Fully Kiosk only supports commands via HTTP and thus a virtual device is needed to send commands.

# Changelog

*23175*: New logic to get online status, with each update.
*22363*: Fixes for *prism_solar*; *owntracks_sensor* now fully supports *notHomeRegionName*, better handling of region transitions.

# Support

Please post to the [SmartHome Community](https://smarthome.community/) and tag me (@therealdb).