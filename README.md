# OCPP & Slimmelezer+ PV Loadbalancing
<a href="https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fwoopstar%2Focpp_slimmelezer_pv_loadbalacer%2Fblob%2Fmain%2Focpp_slimmelezer_pv_loadbalacer.yaml" target="_blank"><img src="https://my.home-assistant.io/badges/blueprint_import.svg" alt="Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled." /></a><br><br><br>
Blueprint for OCPP & Slimmelezer+ PV Loadbalancing

This blueprint makes it possible to tune the charging power of your OCPP charger to match your excess power from solar production.

Blueprint is custom made for OCPP and Slimmelezer+ but will work with other brands of inverters and power messauring devices as long as it can provide the sensors required below is working.

<br>
<b><h2>Important!</h2></b><br>
Before you use this blueprint, make sure you have a OCPP Charger and Slimmelezer+ integrations fully up & running. <br>
<br>
Link to <b>OCPP</b> Integration: https://github.com/lbbrhzn/ocpp <br>
Link to <b>Slimmelezer+</b> https://www.zuidwijk.com/product/slimmelezer-plus/<br>
Link to <b>ha-average</b> https://github.com/Limych/ha-average
<br>
The DSMR protocol used by Slimmelezer+ does not provide a sensor for actual power with negative values for export and positive values for import. Use the following template sensor to create one:
<br><br>

```yaml
- sensor:
  - name: Power Grid Active Power
    unique_id: grid_active_power
    unit_of_measurement: "W"
    device_class: power
    state: >-
      {{ ((states('sensor.power_consumed') | float - states('sensor.power_produced') | float) * 1000) | round }}
```
