---
title: "Template Light"
description: "Instructions on how to integrate Template Lights into Home Assistant."
ha_category:
  - Light
ha_release: 0.46
ha_iot_class: Local Push
ha_quality_scale: internal
ha_domain: template
---

The `template` platform creates lights that combine integrations and provides the
ability to run scripts or invoke services for each of the on, off, and
brightness commands of a light.

To enable Template Lights in your installation, add the following to your
`configuration.yaml` file:

{% raw %}

```yaml
# Example configuration.yaml entry
light:
  - platform: template
    lights:
      theater_lights:
        friendly_name: "Theater Lights"
        level_template: "{{ state_attr('sensor.theater_brightness', 'lux')|int }}"
        value_template: "{{ state_attr('sensor.theater_brightness', 'lux')|int > 0 }}"
        temperature_template: "{{states('input_number.temperature_input') | int}}"
        color_template: "({{states('input_number.h_input') | int}}, {{states('input_number.s_input') | int}})"
        turn_on:
          service: script.theater_lights_on
        turn_off:
          service: script.theater_lights_off
        set_level:
          service: script.theater_lights_level
          data:
            brightness: "{{ brightness }}"
        set_temperature:
          service: input_number.set_value
          data:
            value: "{{ color_temp }}"
            entity_id: input_number.temperature_input
        set_white_value:
          service: input_number.set_value
          data:
            value: "{{ white_value }}"
            entity_id: input_number.white_value_input
        set_color:
          - service: input_number.set_value
            data:
              value: "{{ h }}"
              entity_id: input_number.h_input
          - service: input_number.set_value
            data:
              value: "{{ s }}"
              entity_id: input_number.s_input
```

{% endraw %}

{% configuration %}
  lights:
    description: List of your lights.
    required: true
    type: map
    keys:
      friendly_name:
        description: Name to use in the frontend.
        required: false
        type: string
      unique_id:
        description: An ID that uniquely identifies this light. Set this to a unique value to allow customization through the UI.
        required: false
        type: string
      value_template:
        description: Defines a template to get the state of the light.
        required: false
        type: template
        default: optimistic
      level_template:
        description: Defines a template to get the brightness of the light.
        required: false
        type: template
        default: optimistic
      temperature_template:
        description: Defines a template to get the color temperature of the light.
        required: false
        type: template
        default: optimistic
      white_value_template:
        description: Defines a template to get the white value of the light.
        required: false
        type: template
        default: optimistic
      color_template:
        description: Defines a template to get the color of the light. Must render a tuple (hue, saturation)
        required: false
        type: template
        default: optimistic
      icon_template:
        description: Defines a template for an icon or picture, e.g.,  showing a different icon for different states.
        required: false
        type: template
      availability_template:
        description: Defines a template to get the `available` state of the component. If the template returns `true`, the device is `available`. If the template returns any other value, the device will be `unavailable`. If `availability_template` is not configured, the component will always be `available`.
        required: false
        type: template
        default: true
      turn_on:
        description: Defines an action to run when the light is turned on.
        required: true
        type: action
      turn_off:
        description: Defines an action to run when the light is turned off.
        required: true
        type: action
      set_level:
        description: Defines an action to run when the light is given a brightness command.
        required: false
        type: action
      set_temperature:
        description: Defines an action to run when the light is given a color temperature command.
        required: false
        type: action
      set_white_value:
        description: Defines an action to run when the light is given a white value command.
        required: false
        type: action
      set_color:
        description: Defines an action to run when the light is given a color command.
        required: false
        type: action
{% endconfiguration %}

## Considerations

If you are using the state of a platform that takes extra time to load, the
Template Light may get an `unknown` state during startup. This results
in error messages in your log file until that platform has completed loading.
If you use `is_state()` function in your template, you can avoid this situation.
For example, you would replace
{% raw %}`{{ states.switch.source.state == 'on' }}`{% endraw %}
with this equivalent that returns `true`/`false` and never gives an unknown
result:
{% raw %}`{{ is_state('switch.source', 'on') }}`{% endraw %}

## Examples

In this section you will find some real-life examples of how to use this light.

### Theater Volume Control

This example shows a light that is actually a home theater's volume. This
component gives you the flexibility to provide whatever you'd like to send as
the payload to the consumer including any scale conversions you may need to
make; the [Media Player component](/integrations/media_player/) needs a floating
point percentage value from `0.0` to `1.0`.

{% raw %}

```yaml
light:
  - platform: template
    lights:
      theater_volume:
        friendly_name: "Receiver Volume"
        value_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              off
            {% else %}
              on
            {% endif %}
          {% else %}
            off
          {% endif %}
        turn_on:
          service: media_player.volume_mute
          data:
            entity_id: media_player.receiver
            is_volume_muted: false
        turn_off:
          service: media_player.volume_mute
          data:
            entity_id: media_player.receiver
            is_volume_muted: true
        set_level:
          service: media_player.volume_set
          data:
            entity_id: media_player.receiver
            volume_level: "{{ (brightness / 255 * 100)|int / 100 }}"
        level_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {{ (state_attr('media_player.receiver', 'volume_level')|float * 255)|int }}
          {% else %}
            0
          {% endif %}
```

{% endraw %}

### Change The Icon

This example shows how to change the icon based on the light state.

{% raw %}

```yaml
light:
  - platform: template
    lights:
      theater_volume:
        friendly_name: "Receiver Volume"
        value_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              off
            {% else %}
              on
            {% endif %}
          {% else %}
            off
          {% endif %}
        icon_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              mdi:lightbulb-off
            {% else %}
              mdi:lightbulb-on
            {% endif %}
          {% else %}
            mdi:lightbulb-off
          {% endif %}
        turn_on:
          service: media_player.volume_mute
          data:
            entity_id: media_player.receiver
            is_volume_muted: false
        turn_off:
          service: media_player.volume_mute
          data:
            entity_id: media_player.receiver
            is_volume_muted: true
```

{% endraw %}

### Change The Entity Picture

This example shows how to change the entity picture based on the light state.

{% raw %}

```yaml
light:
  - platform: template
    lights:
      theater_volume:
        friendly_name: "Receiver Volume"
        value_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              off
            {% else %}
              on
            {% endif %}
          {% else %}
            off
          {% endif %}
        icon_template: >-
          {% if is_state('media_player.receiver', 'on') %}
            {% if state_attr('media_player.receiver', 'is_volume_muted') %}
              /local/lightbulb-off.png
            {% else %}
              /local/lightbulb-on.png
            {% endif %}
          {% else %}
            /local/lightbulb-off.png
          {% endif %}
        turn_on:
          service: media_player.volume_mute
          data:
            entity_id: media_player.receiver
            is_volume_muted: false
        turn_off:
          service: media_player.volume_mute
          data:
            entity_id: media_player.receiver
            is_volume_muted: true
```

{% endraw %}
