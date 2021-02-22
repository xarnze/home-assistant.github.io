---
title: Pi-hole
description: Instructions on how to integrate Pi-hole with Home Assistant.
ha_category:
  - System Monitor
  - Sensor
  - Switch
ha_iot_class: Local Polling
ha_config_flow: true
ha_release: 0.28
ha_codeowners:
  - '@fabaff'
  - '@johnluetke'
  - '@shenxn'
ha_domain: pi_hole
ha_platforms:
  - binary_sensor
  - sensor
  - switch
---

The `pi_hole` integration allows you to retrieve statistics and interact with a [Pi-hole](https://pi-hole.net/) system.

{% include integrations/config_flow.md %}

## Services

The platform provides the following services to interact with your Pi-hole. Use switch entities when calling the services.

_Note: Switch entity requires `api_key` to be configured._

### Service `pi_hole.disable`

Disables configured Pi-hole(s) for the specified amount of time.

| Service data attribute | Required | Type | Description |
| ---------------------- | -------- | -------- | ----------- |
| `entity_id` | `False` | string | Target switch entity. Use `all` to target all Pi-hole services |
| `duration` | `True` | timedelta | Time for which Pi-hole should be disabled |
