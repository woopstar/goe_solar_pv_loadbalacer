# go-e MQTT & Huawei Solar PV/EV Loadbalancing
<a href="https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fwoopstar%2Fgoe_huawei_pv_loadbalacer%2Fblob%2Fmain%2Fgoe_huawei_pv_loadbalacer.yaml" target="_blank"><img src="https://my.home-assistant.io/badges/blueprint_import.svg" alt="Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled." /></a><br><br><br>
Blueprint for go-e MQTT & Huawei Solar PV/EV Loadbalancing

This blueprint makes it possible to tune the charging power of your go-e charger to match your excess power from solar production.

Blueprint is custom made for go-e mqtt and Huawei Solar but will work with other brands of inverters and power messauring devices as long as it can provide the sensors required below is working.

With inspiration from https://github.com/MadsGadeberg/EaseeLoadBalancer

<br>
<b><h2>Important!</h2></b><br>
Before you use this blueprint, make sure you have the Sun integration, ha-average, go-e mqtt Charger and Huawei Solar Integrations fully up & running. <br>
<br>
Link to <b>Sun</b> Integration: https://www.home-assistant.io/integrations/sun/  <br>
Link to <b>go-e MQTT</b> Integration: https://github.com/syssi/homeassistant-goecharger-mqtt/ <br>
Link to <b>Huawei Solar Integration</b> https://github.com/wlcrs/huawei_solar/ <br>
Link to <b>ha-average</b> https://github.com/Limych/ha-average <br><br>

The go-e charger does not have a switch to start and stop charging, so the following template switch must be created (remember to change your charger id from 222819 to xxxxxx):<br>


```yaml
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

<br>

You will also need to create the following template sensors to calculate how much excess power you currently produce. To  make sure you do not switch too often, we create a median over 2 minutes for the sensor (remember to change your charger id from 222819 to xxxxxx)::<br><br>

```yaml
- platform: template
  sensors:
    huawei_power_available_for_charging:
      friendly_name: "Huawei Power Available For Charging"
      unique_id: "huawei_power_available_for_charging"
      unit_of_measurement: "W"
      device_class: power
      value_template: >-
        {% set powerAvailable = states('sensor.go_echarger_222819_nrg_12') | float(0) + states('sensor.power_meter_active_power') | float(0) + states('sensor.battery_charge_discharge_power')  | float(0) %}
        {% set powerAvailable = (powerAvailable - states('number.battery_maximum_charging_power') | int(5000)) if states('sensor.battery_state_of_capacity') | int(5) < ( states('number.battery_end_of_charge_soc') | int(100) * 0.98 ) else powerAvailable %}
        {{ powerAvailable | float(0) }}

- platform: average
  name: "Mean Huawei Power Available For Charging 2 min"
  unique_id: mean_huawei_power_available_for_charging_2_min
  entities:
    - sensor.huawei_power_available_for_charging
  duration:
    minutes: 2
```
