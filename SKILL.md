---
name: Home Assistant Expert (Lite)
description: Give Claude working knowledge of Home Assistant automation YAML — write basic automations, understand triggers and conditions, look up entity ID domains.
category: home-automation
version: 1.0.0
author: jazz-systems
license: MIT
---

<!-- Jazz Systems HA Expert Lite v1.0 | Licensed for personal use only. Not for resale or redistribution. -->

# Home Assistant Expert (Lite)

## Purpose
Give Claude working knowledge of Home Assistant automation YAML so it can write basic automations, explain triggers and conditions, and understand entity IDs — without needing to look anything up.

**Want Jinja2 templates, integration deep-dives (Zigbee2MQTT, Z-Wave, Ecobee, ESPHome), Lovelace dashboard cards, and a full debugging protocol? See Home Assistant Expert Pro.**

## When to Invoke This Skill
- Writing simple automations from a plain English description
- Understanding how triggers, conditions, and actions work
- Asking what entity ID domain to use for a given device type
- Getting a working YAML snippet to paste into HA

---

## Entity ID Domains
`light`, `switch`, `sensor`, `binary_sensor`, `fan`, `lock`, `climate`, `media_player`, `camera`, `cover`, `vacuum`, `person`, `input_boolean`, `input_text`, `input_number`, `input_select`, `automation`, `script`, `scene`, `notify`

**Naming convention:** `<domain>.<area>_<device_type>` — e.g. `light.living_room_ceiling`, `binary_sensor.front_door_contact`

---

## Automation Structure

```yaml
automation:
  alias: "Descriptive Name"
  mode: single        # single | restart | queued | parallel
  trigger:
    - platform: state
      entity_id: binary_sensor.front_door_contact
      to: "on"
  condition:
    - condition: time
      after: "07:00:00"
      before: "22:00:00"
  action:
    - service: light.turn_on
      target:
        entity_id: light.foyer
      data:
        brightness_pct: 80
```

---

## Common Triggers

```yaml
# State change
- platform: state
  entity_id: binary_sensor.motion_sensor
  to: "on"
  for: "00:00:30"      # hold state for 30s before firing

# Time
- platform: time
  at: "07:30:00"

# Sun
- platform: sun
  event: sunset
  offset: "-00:30:00"  # 30 min before sunset

# Numeric threshold
- platform: numeric_state
  entity_id: sensor.humidity
  above: 70
```

---

## Common Conditions

```yaml
- condition: state
  entity_id: input_boolean.guest_mode
  state: "off"

- condition: time
  after: "08:00:00"
  before: "22:00:00"
  weekday: [mon, tue, wed, thu, fri]

- condition: numeric_state
  entity_id: sensor.humidity
  above: 50
  below: 80
```

---

## Common Actions

```yaml
# Turn on a light
- service: light.turn_on
  target:
    entity_id: light.kitchen
  data:
    brightness_pct: 100

# Delay
- delay: "00:05:00"

# Send notification
- service: notify.mobile_app_marks_iphone
  data:
    title: "Alert"
    message: "Front door opened"

# If/else (choose)
- choose:
    - conditions:
        - condition: state
          entity_id: input_boolean.away_mode
          state: "on"
      sequence:
        - service: alarm_control_panel.alarm_arm_away
          target:
            entity_id: alarm_control_panel.home
  default:
    - service: alarm_control_panel.alarm_arm_home
      target:
        entity_id: alarm_control_panel.home
```

---

## Execution Protocol

When asked to write an automation:
1. Identify trigger, conditions (if any), and action
2. Use `mode: single` unless the user says it should be re-triggerable (use `restart`) or must queue (use `queued`)
3. Write complete YAML — never use placeholders like `<entity_id>`; ask if you don't know the exact entity ID
4. Tell the user where to add it: Settings → Automations → New Automation → Edit as YAML, or paste into `automations.yaml`
