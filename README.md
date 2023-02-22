# go-E & Slimmelezer+ PV Loadbalancing
<a href="https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fwoopstar%2Fgoe_slimmelezer_pv_loadbalacer%2Fblob%2Fmain%2Fgoe_slimmelezer_pv_loadbalacer.yaml" target="_blank"><img src="https://my.home-assistant.io/badges/blueprint_import.svg" alt="Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled." /></a><br><br><br>
Blueprint for go-e & Slimmelezer+ PV Loadbalancing

This blueprint makes it possible to tune the charging power of your go-e charger to match your excess power from solar production.

Blueprint is custom made for go-e mqtt and Slimmelezer+ but will work with other brands of inverters and power messauring devices as long as it can provide the sensors required below is working.

<br>
<b><h2>Important!</h2></b><br>
Before you use this blueprint, make sure you have a go-e mqtt Charger and Slimmelezer+ integrations fully up & running. <br>
<br>
Link to <b>go-e MQTT</b> Integration: [https://github.com/lbbrhzn/ocpp](https://github.com/syssi/homeassistant-goecharger-mqtt) <br>
Link to <b>Slimmelezer+</b> https://www.zuidwijk.com/product/slimmelezer-plus/<br>
Link to <b>ha-average</b> https://github.com/Limych/ha-average
<br>
The DSMR protocol used by Slimmelezer+ does not provide a sensor for actual power with negative values for export and positive values for import. Use the following template sensor to create:
<br><br>

```yaml
- sensor:
  - platform: template
    sensors:
      grid_active_power:
        friendly_name: "Power Grid Active Power"
        unique_id: "grid_active_power"
        unit_of_measurement: "W"
        device_class: power
        value_template: >-
          {{ ((states('sensor.power_consumed') | float - states('sensor.power_produced') | float) * 1000) | round(3, default=0) }}

  - platform: average
    name: "Mean Power Grid Active Power"
    unique_id: mean_grid_active_power
    entities:
      - sensor.power_grid_active_power
    duration:
      minutes: 2

  - platform: template
    sensors:
      grid_power_available_for_charging:
        friendly_name: "Power Grid Available For Charging"
        unique_id: "grid_power_available_for_charging"
        unit_of_measurement: "W"
        device_class: power
        value_template: >-
          {% set power = ((states('sensor.power_consumed') | float - states('sensor.power_produced') | float) * 1000) | round(3, default=0) %}
          {{ power if power < 0 else 0 }}

  - platform: average
    name: "Mean Power Grid Available For Charging"
    unique_id: mean_grid_power_available_for_charging
    entities:
      - sensor.grid_power_available_for_charging
    duration:
      minutes: 5
```

You will also need a custom switch to turn on and off charging on the charger, since go-e mqtt does not have that implemented:

```yaml
switch:
  - platform: template
    switches:
      go_echarger_222819_allow_charging:
        friendly_name: "Allow Charging"
        unique_id: go_echarger_222819_allow_charging
        value_template: "{{ is_state('sensor.go_echarger_222819_frc', 'Charge') }}"
        turn_on:
          service: select.select_option
          target:
            entity_id: select.go_echarger_222819_frc
          data:
            option: "Charge"
        turn_off:
          service: select.select_option
          target:
            entity_id: select.go_echarger_222819_frc
          data:
            option: "Don't charge"
        icon_template: >-
          {% if is_state('sensor.go_echarger_222819_frc', 'Charge') %}
            mdi:power-plug
          {% else %}
            mdi:power-plug-off
          {% endif %}
```
