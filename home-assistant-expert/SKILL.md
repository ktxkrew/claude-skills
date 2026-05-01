---
name: Home Assistant Expert
description: Deep Home Assistant expertise for Claude — write automations, debug entities, build Lovelace dashboards, and use Jinja2 templates from plain English descriptions.
category: home-automation
version: 1.0.0
author: jazz-systems
license: MIT
---

# Home Assistant Expert
<!-- Jazz Systems HA Expert v1.0 | jazz-systems.io | Licensed for personal use only. Not for resale or redistribution. -->

## Purpose
Transform Claude into a deep Home Assistant expert capable of writing automations, diagnosing entity issues, building dashboards, and architecting smart home logic from plain English descriptions — without needing to look anything up.

## When to Invoke This Skill
- Writing or debugging YAML automations, scripts, or scenes
- Diagnosing why an entity, integration, or automation isn't working
- Designing Lovelace dashboard layouts and cards
- Choosing between integrations for a given device or use case
- Asking how to template, condition, or trigger anything in HA
- Building a new smart home feature from scratch

---

## Core Knowledge: Home Assistant Architecture

### Config Structure
```
/config/
  configuration.yaml      — main config, includes other files
  automations.yaml        — automation list (or automations/ directory)
  scripts.yaml            — script definitions
  scenes.yaml             — scene definitions
  secrets.yaml            — credentials (never commit this)
  custom_components/      — HACS integrations
  www/                    — static files served at /local/
```

### Entity ID Naming Convention
`<domain>.<area>_<device_type>_<attribute>`
Examples:
- `light.living_room_ceiling` — not `light.philips_hue_1`
- `binary_sensor.front_door_contact` — not `binary_sensor.zigbee_abc123`
- `sensor.master_bathroom_humidity` — descriptive, not device-named

Domains: `light`, `switch`, `sensor`, `binary_sensor`, `input_boolean`, `input_text`, `input_number`, `input_select`, `input_datetime`, `fan`, `lock`, `alarm_control_panel`, `climate`, `media_player`, `camera`, `cover`, `vacuum`, `person`, `zone`, `automation`, `script`, `scene`, `notify`, `device_tracker`, `counter`, `timer`

---

## Core Knowledge: Automation Structure

```yaml
automation:
  alias: "Descriptive Name"
  description: "One-line explanation"
  mode: single              # single | restart | queued | parallel
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

### Trigger Types (most common)
```yaml
# State change
- platform: state
  entity_id: binary_sensor.motion_sensor
  to: "on"
  for: "00:00:30"          # optional: must hold state for 30s

# Time
- platform: time
  at: "07:30:00"

# Sun
- platform: sun
  event: sunset
  offset: "-00:30:00"      # 30 min before sunset

# Template (arbitrary condition becomes true)
- platform: template
  value_template: "{{ states('sensor.temperature') | float > 80 }}"

# Numeric state
- platform: numeric_state
  entity_id: sensor.humidity
  above: 70

# MQTT
- platform: mqtt
  topic: "home/sensor/temp"
  payload: "on"

# Webhook
- platform: webhook
  webhook_id: my_webhook_id
```

### Condition Types
```yaml
- condition: state
  entity_id: input_boolean.guest_mode
  state: "off"

- condition: numeric_state
  entity_id: sensor.humidity
  above: 50
  below: 80

- condition: time
  after: "08:00:00"
  before: "22:00:00"
  weekday: [mon, tue, wed, thu, fri]

- condition: template
  value_template: "{{ is_state('person.mark', 'home') }}"

- condition: or
  conditions:
    - condition: state
      entity_id: binary_sensor.door_1
      state: "on"
    - condition: state
      entity_id: binary_sensor.door_2
      state: "on"
```

### Action Types
```yaml
# Call a service
- service: light.turn_on
  target:
    entity_id: light.kitchen
  data:
    brightness_pct: 100
    color_temp_kelvin: 4000

# Delay
- delay: "00:05:00"

# Wait for state
- wait_for_trigger:
    - platform: state
      entity_id: binary_sensor.motion
      to: "off"
  timeout: "00:30:00"
  continue_on_timeout: false

# Set input helper
- service: input_boolean.turn_on
  target:
    entity_id: input_boolean.sleep_mode

# Send notification
- service: notify.mobile_app_marks_iphone
  data:
    title: "Alert"
    message: "Front door opened"

# Choose (if/else)
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

# Repeat
- repeat:
    count: 3
    sequence:
      - service: light.toggle
        target:
          entity_id: light.bedroom
      - delay: "00:00:02"

# Variables (set within action for reuse)
- variables:
    temp: "{{ states('sensor.outdoor_temp') | float }}"
```

---

## Core Knowledge: Jinja2 Templates

### Getting State and Attributes
```jinja2
{{ states('sensor.temperature') }}                    # raw state string
{{ states('sensor.temperature') | float(0) }}         # convert, default 0 if unavailable
{{ state_attr('light.bedroom', 'brightness') }}       # attribute
{{ is_state('binary_sensor.door', 'on') }}            # boolean
{{ is_state_attr('fan.bedroom', 'percentage', 100) }} # attribute match
```

### Context and Time
```jinja2
{{ now() }}                   # current datetime
{{ now().hour }}              # current hour (0-23)
{{ now().strftime('%H:%M') }} # formatted time
{{ as_timestamp(states('sensor.last_motion')) | int }}
{{ (now() - states.sensor.x.last_changed).total_seconds() | int }}
```

### Conditional Logic
```jinja2
{% if is_state('binary_sensor.door', 'on') %}
  Door is open
{% elif is_state('binary_sensor.door', 'off') %}
  Door is closed
{% else %}
  Unknown
{% endif %}

{{ 'on' if is_state('light.kitchen', 'on') else 'off' }}
```

### Math and Filters
```jinja2
{{ (states('sensor.temp') | float * 9/5 + 32) | round(1) }} # C to F
{{ [50, states('sensor.brightness') | int] | max }}          # clamp minimum
{{ states_attr('light.x', 'brightness') / 255 * 100 | int }} # brightness_pct
```

---

## Core Knowledge: Common Integrations

### Zigbee2MQTT
- Coordinator: Sonoff Zigbee dongle or similar, config in `/config/zigbee2mqtt/`
- Entities auto-created with friendly names from device name in Z2M
- MQTT topics: `zigbee2mqtt/<device_name>/set` to control
- Pairing: put coordinator in pairing mode via Z2M UI, then pair device
- Common issue: device drops off mesh → add Zigbee router plugs (IKEA Tradfri outlet, Sonoff S31 Lite)

### Z-Wave JS
- Requires Z-Wave USB stick, config via Z-Wave JS integration
- Inclusion: use Z-Wave JS UI → Add Device → pair device within 30s
- Entities: auto-created, may need renaming after inclusion
- Common issue: dead nodes → Heal Network or exclude/re-include

### Ecobee
- Integration: Settings → Integrations → Ecobee
- `climate.ecobee_thermostat` — main entity
- `hvac_mode`: heat, cool, heat_cool, off
- Presets: home, away, sleep, custom
- Smart Home/Away disables schedule — disable in Ecobee app if automating presence

### ESPHome
- Flash ESP8266/ESP32 via ESPHome Dashboard (add-on)
- Auto-discovered by HA via mDNS
- Secrets managed in ESPHome's secrets.yaml (separate from HA)
- Entities: created per sensor/switch defined in device YAML

### HACS (Custom Integrations)
- Install via: Settings → Add-ons → HACS → Frontend or Integration
- Requires GitHub token on first setup
- Popular: browser_mod, mushroom cards, auto-entities, card-mod, Alexa Media Player

---

## Core Knowledge: Lovelace / Dashboard

### Card Types
```yaml
# Entities card (list)
type: entities
entities:
  - entity: light.living_room
  - entity: switch.fan
title: Living Room

# Gauge
type: gauge
entity: sensor.humidity
min: 0
max: 100
severity:
  green: 0
  yellow: 60
  red: 80

# History graph
type: history-graph
entities:
  - entity: sensor.temperature
hours_to_show: 24

# Conditional (show/hide based on state)
type: conditional
conditions:
  - entity: binary_sensor.motion
    state: "on"
card:
  type: entity
  entity: binary_sensor.motion

# Button
type: button
entity: script.bedtime_routine
name: Bedtime
tap_action:
  action: call-service
  service: script.turn_on
  target:
    entity_id: script.bedtime_routine
```

---

## Step-by-Step Execution Protocol

### When asked to write an automation:
1. Identify: what triggers it, any conditions, what it does
2. Choose correct `mode` — `single` for most; `restart` if re-triggerable; `queued` if must complete each run
3. Write YAML exactly — no placeholders, no `<entity_id>` substitutions. Ask for exact entity IDs if not provided.
4. Include a `description` field
5. Mention how to add it: UI Editor (Settings → Automations → + New → Edit as YAML) or `automations.yaml`

### When asked to debug:
1. Ask for: entity state, last changed time, automation trace (Settings → Automations → automation → Traces)
2. Check: trigger fired? Conditions passed? Action errored?
3. Check logbook for the entity around the time it should have fired
4. If unavailable state: check integration health, restart integration before restarting HA

### When asked about entity not appearing:
1. Check Developer Tools → States — is it there but named differently?
2. Check integration is loaded (Settings → Devices & Services)
3. Check `homeassistant.log` for errors at startup
4. Try: Settings → Server Controls → Reload All YAML (for config) or restart

---

## Skill Boundaries
This skill does NOT handle:
- Writing custom integrations (Python/HACS components) — that requires Engineering context
- Network configuration, VLANs, firewall rules
- Third-party cloud service APIs (Alexa, Google Home backend) beyond what HA integration covers

For these, escalate to a developer or reference the HA developer documentation.
