# reactor-mqtt-contrib
Additional MQTT templates for Reactor.

# Installation
See Reactor doc for detailed installation.
Copy the contents (or the file, if you don't have other templates) under Reactor's config directory.

# Templates

| Template ID | Device | Capabilities |
| ------------- | ------------- | ------------- |
| shelly_hem | Shelly Home Energy Meter | power_sensor, energy_sensor, voltage_sensor, current_sensor, power_factor_sensor, x_energy_sensor_exported |
| shelly_binary | Shelly with detached inputs, mapped as binary sensor | binary_sensor |
| shelly_exttemperature | Shelly with detached inputs, mapped as binary sensor | binary_sensor |
| shelly_exttemperature | Shelly with external temperature sensor | temperature_sensor |
| shelly_exthumidity | Shelly with external humidity sensor | humidity_sensor |
| tasmota_sensor_pressure | Tasmota with Pressure sensor | value_sensor |
| tasmota_sensor_illuminance | Tasmota with Illuminance sensor | light_sensor |