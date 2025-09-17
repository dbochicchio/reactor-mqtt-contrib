# reactor-mqtt-contrib

This repository contains additional MQTT templates for [Reactor](https://reactor.toggledbits.com/docs/).

# Installation

1. Download the ZIP file from the releases.
2. Save it under *reactor/config/mqtt_templates*. If the *mqtt_templates* directory does not exist, create it.
3. If you prefer to have them in a directory, create one and unzip the files there. Reactor will search for templates in any ZIP or YAML file in this location.
4. After copying the files, restart Reactor. A restart is required each time you update the files.

# How to Update

To update from a previous version, delete all existing files. Then, follow the installation procedure again.

# Templates

All templates support *query*/*init* commands, and their state will be updated at startup. The *x_mqtt.poll* action can be used to poll specific devices in reactions. The *wifi_status* capability is supported where information is provided.

## Shelly Gen1

Shelly Gen1 templates are mature, and most device types are supported. These devices have a special *x_shelly_gen1* capability, with *update_firwmare* command to issue firmware updates. A special *firwmare_available* (bool) attribute is provided to signal when a firmware update is present.

| Template ID | Device | Capabilities | Parameters |
| ------------- | ------------- | ------------- | ------------- |
| shelly_hem | Shelly Home Energy Meter (Shelly EM, Shelly 3EM) | power_sensor, energy_sensor, voltage_sensor, current_sensor, power_factor_sensor, x_energy_sensor_exported, wifi_status | topic, channel |
| shelly_relay_simple | Shelly relay (simple version) | switch, toggle, wifi_status | topic, channel |
| shelly_relay_simple_reversed | Shelly relay with reversed status (simple version) | switch, toggle, wifi_status | topic, channel |
| shelly_relay_power | Shelly relay with power meter | switch, toggle, power_sensor, energy_sensor, voltage_sensor, current_sensor, power_factor_sensor, wifi_status | topic, channel |
| shelly_binary | Shelly with detached inputs, mapped as binary sensor, wifi_status | binary_sensor | topic, channel |
| shelly_cover | Shelly 2.5 in cover mode | cover, position, power_switch, toggle, power_sensor, energy_sensor, wifi_status | topic |
| shelly_dimmer | Shelly dimmer | dimming, power_switch, toggle, power_sensor, energy_sensor, wifi_status | topic, channel |
| shelly_exttemperature | Shelly with detached inputs, mapped as binary sensor | binary_sensor, wifi_status | topic, channel |
| shelly_exthumidity | Shelly with external humidity sensor | humidity_sensor, wifi_status | topic, channel |
| shelly_scenecontroller | Shelly as button or scene controller, use with detached inputs or to handle single, double, triple clicks, or long push | button, scene_activation, wifi_status | topic, channel |
| shelly_button1 | Shelly Button 1 | button, battery_power, battery_maintenance, wifi_status | topic, channel |
| shelly_dw2 | Shelly Door/Window 2 | door_sensor, light_sensor, tilt_sensor, motion_sensor, battery_power, wifi_status | topic |
| shelly_uni_adc | Shelly UNI ADC | value_sensor, wifi_status | topic |

## Shelly Gen3

Shelly Gen3 templates are a work in progress. I currently have only a few of them. Feel free to ask for a specific template (be sure to provide MQTT logs).

| Template ID | Device | Capabilities | Parameters |
| ------------- | ------------- | ------------- | ------------- |
| shelly_relay_gen3 | Shelly relay (simple version) | switch, toggle, wifi_status | topic, channel |
| shelly_relay_power_gen3 | Shelly relay with power meter | switch, toggle, power_sensor, energy_sensor, voltage_sensor, current_sensor, wifi_status | topic, channel |
| shelly_dimmer_power_gen3 | Shelly dimmer with power meter | switch, toggle, dimming, power_sensor, energy_sensor, voltage_sensor, current_sensor, wifi_status | topic, channel |
| shelly_3em_gen3| Shelly 3 EM (aggregate channel) |power_sensor, energy_sensor, voltage_sensor, current_sensor, wifi_status | topic |
| shelly_hem_gen3| Shelly EM (single channel) |power_sensor, energy_sensor, voltage_sensor, current_sensor, wifi_status | topic |

## Tasmota

| Template ID | Device | Capabilities | Parameters |
| ------------- | ------------- | ------------- | ------------- |
| tasmota_sensor_pressure | Tasmota with pressure sensor | value_sensor, wifi_status | topic, source |
| tasmota_sensor_illuminance | Tasmota with illuminance sensor | light_sensor, wifi_status | topic, source |

## Others

| Template ID | Device | Capabilities | Parameters |
| ------------- | ------------- | ------------- | ------------- |
| fullykiosk | [Fully Kiosk](https://www.fully-kiosk.com/). See additional configuration for info. | string_sensor, binary_sensor, battery_power, battery_maintenance, dimming, wifi_status | topic |
| owntracks_sensor | OwnTracks Sensor with multiple information (position, current region, device battery). See additional configuration for info. | string_sensor, binary_sensor, battery_power, battery_maintenance, location | prefix, topic, homeRegionName, notHomeRegionName |
| prism_solar_charger, prism_solar_session | Prism Solar EV Charger from [Silla Industries](https://silla.industries/en/docs/prism/prism-use-and-maintenance/). See additional configuration for info. | ev_charger, power_switch, toggle, power_sensor, energy_sensor, voltage_sensor, current_sensor | topic, channel |
| homekey_esp32| [HomeKey-ESP32](https://github.com/rednblkx/HomeKey-ESP32). See additional configuration for info. | lock, tag, wifi_status | auth_topic, state_topic, state_set_topic, lwt_topic |

# Configuration

Let's say you want to map a Shelly EM. In your *reactor.yaml*, under *controllers:*, search for *mqtt*, then *config*, and add:

```yaml
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

Where *shelly-solar* is the device name (configured in your Shelly as the MQTT topic) and *channel* is the index (Shelly EM supports 2 clamps). Refer to the table above for other parameters.

## Additional Configuration

### Dual Switch Devices

Dual switch devices, such as Shelly 2 or 2.5, should be mapped using a separate device for each channel. Specify the *channel* parameters accordingly (starting from 0).

### Scene Controllers

Many devices offer scene controller capabilities. Map a new device using *shelly_scenecontroller* as a template. Specify the *channel* parameters accordingly for multi-button devices (Shelly 2.5, i3, etc.), always starting from 0 as the first channel.

### Shelly Uni

Shelly Uni offers two inputs and two outputs. Map it using *shelly_relay_simple* (with *channel* as 0 or 1) for outpouts and *shelly_binary* (with *channel* as 0 or 1) for inputs. This will give you four devices: two are mapping the inputs and two are mapping the outputs.
For temperature/humidity sensors, map a new device using *shelly_exttemperature* or *shelly_exthumidity*, setting the appropriate channels. This typically results in 4-5 devices mapped, giving you full control over what is represented. Use *shelly_uni_adc* for ADC measurements.

### OwnTracks

This template creates a device with:
 - a string sensor for the current region
 - a binary sensor that's true when the user is at home
 - location info via *location* capability
 - battery info for the device (including percentage and charging status)

Refer to the [OwnTracks documentation](https://owntracks.org/booklet/guide/topics/) for more information on the topic structure.

Suppose your base topic for OwnTracks is `owntracks/daniele/iPhone`. In your *reactor.yaml*, under *controllers:*, search for *mqtt*, and then under *config* add:

```yaml
        ...
        owntracks_daniele: #entity ID - must be unique
          name: "Daniele's Location" # friendly name
          prefix: "daniele"
          topic: "iPhone"
          uses_template: owntracks_sensor
          homeRegionName: "home"
          notHomeRegionName: "not_home"
```

Where `prefix` is the first part of the MQTT topic (*daniele* in this case) and `topic` is the last part, typically the device (*iPhone*). `homeRegionName` is the name of your home region and will be used to update the binary sensor. It's case insensitive. `notHomeRegionName` is the name of the region when not home. If omitted, it defaults to null.

If you have multiple devices to track, repeat the same configuration using a different entity ID.

### Prism Solar

To fully support this EV Charger, map two devices:

```yaml
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

The first device will implement the charger and its commands. The second will show the information (power, current, and energy) for the current charging session. *Channel* is usually 1, unless you have a Prism Solar Duo. In that case, map multiple devices using *prism_solar_charger* as the base template. Neither poll nor LWT are supported by Prism Solar. Although night mode is supported via MQTT messages, it's better to create something with Reactor capabilities due to its cumbersome structure and cloud dependency.

### Fully Kiosk

This device supports these capabilities:
 - binary sensor (for screen on/off)
 - dimming (for screen brightness)
 - battery level and charging status (plugged-in, battery power)
 - string_sensor (to show the current app)

Unfortunately, Fully Kiosk supports commands only via HTTP, so a virtual device is needed to send commands.

#### HomeKey-ESP32

[HomeKey-ESP32](https://github.com/rednblkx/HomeKey-ESP32) is a cool project that gives you an HomeKey without a lock, in a pure DIY manner. See the previous link for info and instructions on how to build the hardware, the software and configure everything. The device will publish its state under MQTT.

The following attributes should be specified in the device configuragion under Reactor's MQTT section:

```yaml
        ...
        # HomeKeys
        homekey_front:
          name: "HomeKey Front"
          uses_template: homekey_esp32
          auth_topic : "homekeys/front/auth"
          state_topic: "homekeys/front/state"
          state_set_topic: "homekeys/front/set_state"
          lwt_topic: "homekey-front/status"
```

You'll get the default MQTT topics from the HomeKey-ESP32 web UI. My advice is to modify them in a similar way as in the example. I use the ```homekeys/%reader%/%event%``` because I have multiple readers and to stay consistent with my style, but anything is possible and the template will accomodate any style.

```tag``` capability will have these attributes:
  - ```device_id```: the reader ID (unique)
  - ```tag_id```: the key ID (unique by home in Apple Home)
  - ```last_scanned```: epoc for last scanned date/time
  - ```tag_type```: HomeKey or NFC (for non HomeKey tags)
  - ```scan_count```: always 1

The HomeKey is a lock under Apple Home, so the device in Reactor will offer you a ```lock``` capability, that it's intended to map the real lock/garage door under Apple Home. By default, the state will not be changed when the card is scanned and your logic will need to be written inside a rule. I have a rule that's setting ```lock.state``` based on the real lock/garage door under Reactor, and another one that's watching for ```tag.device_id```, ```tag.tag_id```, and ```tag.last_scanned``` to determine if the key is valid and the lock/garage door should be opened or not.

# Contribute

If you have MQTT payloads for Shelly Gen3 and want them covered, use the procedure outlined in *Support* to request a template.

# Changelog

 - *26228*: New template for *shelly_3em_gen3*. Fixes for *shelly_hem_gen3*.
 - *25244*: New templates for shelly_hem_gen3, shelly_dimmer_power_gen3. Fixes.
 - *24265*: Fixes for Shelly *energy_sensor*.
 - *24258*: New template for *HomeKey-ESP32*. *power_source* capability for Fully Kiosk. Requires Reactor 24257.
 - *24210*: Bug fixing. New templates for *shelly_relay_power_gen3*.
 - *24161*: new *x_shelly_gen1* capability, with *update_firmware* command.
 - *24153*: Refactoring, removal of *switchbot_switch* template.
 - *24147*: Fixes post 24146.
 - *24146*: New features for MQTTController 24144, refactoring.
 - *24144*: Added support for MQTTController 24144.
 - *24111*: Requires MQTTController v24108. Bug fixes, support for *wifi_status*.
 - *23234*: Shelly's Temperature and Humidity sensors report *999* as value if null. Added a filter to prevent incorrect readings. Added support for *shelly_relay_simple_reversed* and *shelly_uni_adc*.
 - *24002*: New logic for online status with each update.
 - *22363*: Fixes for *prism_solar*; *owntracks_sensor* now fully supports *notHomeRegionName*, better handling of region transitions.

# Support

Please post to the [SmartHome Community](https://smarthome.community/) and tag me (@therealdb).