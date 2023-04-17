# go-e MQTT & Solar Power PV/EV Loadbalancing
<a href="https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fwoopstar%2Fgoe_solar_pv_loadbalacer%2Fblob%2Fmain%2Fgoe_solar_pv_loadbalacer.yaml" target="_blank"><img src="https://my.home-assistant.io/badges/blueprint_import.svg" alt="Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled." /></a><br><br><br>
Blueprint for go-e MQTT & Solar Power PV/EV Loadbalancing

This blueprint makes it possible to tune the charging power of your go-e charger to match your excess power from solar production.

Blueprint is custom made for go-e mqtt and Solar Power but will work with other brands of inverters and power messauring devices as long as it can provide the sensors required below is working.

With inspiration from https://github.com/MadsGadeberg/EaseeLoadBalancer

<br>
<b><h2>Important!</h2></b><br>
Before you use this blueprint, make sure you have the Sun integration, go-e mqtt Charger and Solar Integrations fully up & running. <br>
<br>
Link to <b>Sun</b> Integration: https://www.home-assistant.io/integrations/sun/  <br>
Link to <b>go-e MQTT</b> Integration: https://github.com/syssi/homeassistant-goecharger-mqtt/ <br>
 <br>
And if you use Huawei: <br>
Link to <b>Huawei Solar Integration</b> https://github.com/wlcrs/huawei_solar/ <br>
<br>
The go-e charger does not have a switch to start and stop charging, so the following template switch must be created (remember to change your charger id from 222819 to xxxxxx):<br>
<br>

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

You will also need to create the following template sensors to calculate how much excess power you currently produce. To  make sure you do not switch too often, we create a mean over 2 minutes for the sensor (remember to change your charger id from 222819 to xxxxxx)::<br><br>

For people <b>with</b> battery:

```yaml
- platform: template
  sensors:
    solar_power_available_for_charging:
      friendly_name: "Solar Power Available For Charging"
      unique_id: "solar_power_available_for_charging"
      unit_of_measurement: "W"
      device_class: power
      value_template: >-
        {%- set carCharger = states('sensor.go_echarger_222819_nrg_12') | float(0) %}
        {%- set powerMeter = states('sensor.power_meter_active_power') | float(0) %}
        {%- set batteryCharger = states('sensor.battery_charge_discharge_power') | float(5000) %}
        {%- set batterySoC = states('sensor.battery_state_of_capacity') | int(5) %}
        {%- set batteryTargetSoc = states('number.battery_end_of_charge_soc') | int(100) * 0.98 %}
        {%- set powerAvailable = carCharger + powerMeter + batteryCharger | float(0) %}
        {%- set powerAvailable = (powerAvailable - batteryCharger) if batterySoC < batteryTargetSoc else powerAvailable %}
        {{ 0 if powerAvailable | float(0) > 0 else powerAvailable | float(0) }}
```


For people <b>with-out</b> battery:

```yaml
- platform: template
  sensors:
    solar_power_available_for_charging:
      friendly_name: "Solar Power Available For Charging"
      unique_id: "solar_power_available_for_charging"
      unit_of_measurement: "W"
      device_class: power
      value_template: >-
        {%- set carCharger = states('sensor.go_echarger_222819_nrg_12') | float(0) %}
        {%- set powerMeter = states('sensor.power_meter_active_power') | float(0) %}
        {%- set powerAvailable = carCharger + powerMeter | float(0) %}
        {{ 0 if powerAvailable | float(0) > 0 else powerAvailable | float(0) }}
```

And the mean statistics sensor:

```yaml
- platform: statistics
  name: "Mean Solar Power Available For Charging Over 2 min"
  entity_id: sensor.solar_power_available_for_charging
  unique_id: mean_solar_power_available_for_charging_over_2_min
  state_characteristic: mean
  max_age:
    minutes: 2
```
