# Install Redline after WTG discovery is working

These steps assume WTG is already publishing MQTT telemetry and Home Assistant has already discovered the WTG GPU device.

You should already see a WTG GPU card or WTG GPU entities in Home Assistant before starting this guide.

Redline does not replace WTG discovery. Redline adds Home Assistant template entities and dashboard cards on top of the WTG entities that already exist.

## What you will install

This repo provides two things:

```text
packages/wtg_gpu_redline.yaml
```

A Home Assistant package that creates Redline template entities.

```text
dashboards/redline-basic.yaml
```

A basic dashboard card example that uses those Redline entities.

The package file is a starting template. Copy it into your Home Assistant `/config/packages/` directory, then edit the source entity IDs if your WTG advertised hostname or GPU index is different from the example.

## 1. Confirm your WTG entity IDs

In Home Assistant, open:

```text
Developer Tools -> States
```

Search for:

```text
wtg
```

WTG discovery creates deterministic Home Assistant entity IDs that begin with `wtg_` followed by the advertised hostname. For example, if WTG advertises the hostname `bench`, the entity IDs begin with:

```text
sensor.wtg_bench_
```

For the default single-GPU example, the discovered entities look like this:

```text
sensor.wtg_bench_gpu_0_gpu_0_gpu_utilization
sensor.wtg_bench_gpu_0_gpu_0_memory_controller_utilization
sensor.wtg_bench_gpu_0_gpu_0_power
sensor.wtg_bench_gpu_0_gpu_0_power_limit
sensor.wtg_bench_gpu_0_gpu_0_temperature
sensor.wtg_bench_gpu_0_gpu_0_vram_used
sensor.wtg_bench_gpu_0_gpu_0_vram_total
sensor.wtg_bench_gpu_0_gpu_0_performance_state
```

For another advertised hostname, replace `bench` with that hostname.

Example pattern:

```text
sensor.wtg_<advertised_hostname>_gpu_0_gpu_0_gpu_utilization
sensor.wtg_<advertised_hostname>_gpu_0_gpu_0_memory_controller_utilization
sensor.wtg_<advertised_hostname>_gpu_0_gpu_0_power
sensor.wtg_<advertised_hostname>_gpu_0_gpu_0_power_limit
sensor.wtg_<advertised_hostname>_gpu_0_gpu_0_temperature
sensor.wtg_<advertised_hostname>_gpu_0_gpu_0_vram_used
sensor.wtg_<advertised_hostname>_gpu_0_gpu_0_vram_total
sensor.wtg_<advertised_hostname>_gpu_0_gpu_0_performance_state
```

Keep this browser tab open. You will use these entity IDs when editing the Redline package.

## 2. Create the packages directory

Home Assistant packages live under:

```text
/config/packages/
```

You can create this directory with a file editor add-on or from a Home Assistant terminal.

### Option A: File editor or Studio Code Server

Use one of these Home Assistant add-ons if installed:

```text
File editor
Studio Code Server
Samba share
```

Create the folder:

```text
/config/packages/
```

### Option B: Terminal

If you prefer the command line, install or open a Home Assistant terminal add-on, such as:

```text
Terminal & SSH
Advanced SSH & Web Terminal
```

Run:

```sh
mkdir -p /config/packages
```

A terminal is not required if you are using File editor, Studio Code Server, or Samba.

## 3. Copy the Redline package from this repo

Create this file in Home Assistant:

```text
/config/packages/wtg_gpu_redline.yaml
```

Copy the contents of this repo file into it:

```text
packages/wtg_gpu_redline.yaml
```

Direct repo path:

```text
https://github.com/novovictus/wtg-ha-redline/blob/main/packages/wtg_gpu_redline.yaml
```

Important: before saving, edit the package if your discovered WTG entity IDs are different from the example.

The default example currently references:

```text
sensor.wtg_bench_gpu_0_gpu_0_gpu_utilization
sensor.wtg_bench_gpu_0_gpu_0_memory_controller_utilization
sensor.wtg_bench_gpu_0_gpu_0_power
sensor.wtg_bench_gpu_0_gpu_0_power_limit
sensor.wtg_bench_gpu_0_gpu_0_temperature
sensor.wtg_bench_gpu_0_gpu_0_vram_used
sensor.wtg_bench_gpu_0_gpu_0_vram_total
sensor.wtg_bench_gpu_0_gpu_0_performance_state
```

If your advertised hostname is not `bench`, replace `wtg_bench` with your actual discovered prefix.

For example:

```text
sensor.wtg_bench_gpu_0_gpu_0_power
```

might become:

```text
sensor.wtg_myhostname_gpu_0_gpu_0_power
```

## 4. Enable Home Assistant packages

Open:

```text
/config/configuration.yaml
```

Packages are enabled by adding this block exactly as shown:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

This is not a placeholder and does not need editing. It tells Home Assistant to load every YAML file inside:

```text
/config/packages/
```

The word `packages` at the end is the folder name, not your Redline package name.

If there is no `homeassistant:` block in `configuration.yaml`, add the full block:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

If a `homeassistant:` block already exists, add only this indented line under the existing `homeassistant:` block:

```yaml
  packages: !include_dir_named packages
```

Do not create a second `homeassistant:` block.

Example default Home Assistant configuration before:

```yaml
# Loads default set of integrations. Do not remove.
default_config:

# Load frontend themes from the themes folder
frontend:
  themes: !include_dir_merge_named themes

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml
```

Example after adding packages:

```yaml
# Loads default set of integrations. Do not remove.
default_config:

homeassistant:
  packages: !include_dir_named packages

# Load frontend themes from the themes folder
frontend:
  themes: !include_dir_merge_named themes

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml
```

## 5. Check Home Assistant configuration

Open:

```text
Developer Tools -> YAML
```

Click:

```text
Check configuration
```

Do not restart unless the configuration check passes.

## 6. Restart Home Assistant

After the configuration check passes, restart Home Assistant:

```text
Developer Tools -> YAML -> Restart
```

## 7. Confirm Redline entities exist

After restart, open:

```text
Developer Tools -> States
```

Search for:

```text
redline
```

You should see:

```text
binary_sensor.wtg_bench_redline_sus
sensor.wtg_bench_redline_score
sensor.wtg_bench_redline_state
sensor.wtg_bench_redline_summary
```

If you changed the Redline entity names or unique IDs in the package, your entity IDs may differ.

## 8. Add the dashboard cards

Open the dashboard where you want the gauge.

Select:

```text
Edit dashboard -> Add card
```

Home Assistant shows a list of card types. Scroll all the way to the bottom of the list and select:

```text
Manual
```

Paste this gauge card:

```yaml
type: gauge
entity: sensor.wtg_bench_redline_score
name: WTG GPU Redline
min: 0
max: 100
needle: true
severity:
  green: 0
  yellow: 20
  red: 85
```

Then add another manual card:

```yaml
type: entities
title: WTG GPU Redline
entities:
  - entity: sensor.wtg_bench_redline_state
    name: State
  - entity: sensor.wtg_bench_redline_score
    name: Redline Score
  - entity: binary_sensor.wtg_bench_redline_sus
    name: SUS Override
  - entity: sensor.wtg_bench_redline_summary
    name: Summary
  - entity: sensor.wtg_bench_gpu_0_gpu_0_gpu_utilization
    name: GPU
  - entity: sensor.wtg_bench_gpu_0_gpu_0_memory_controller_utilization
    name: Memory Controller
  - entity: sensor.wtg_bench_gpu_0_gpu_0_power
    name: Power
  - entity: sensor.wtg_bench_gpu_0_gpu_0_power_limit
    name: Power Limit
  - entity: sensor.wtg_bench_gpu_0_gpu_0_temperature
    name: Temperature
  - entity: sensor.wtg_bench_gpu_0_gpu_0_performance_state
    name: Perf State
```

If your WTG source entities are different from the example, edit the entity card too.

## 9. Expected idle result

At idle, the system should normally show:

```text
State: IDLE
SUS Override: off
Redline Score: low
```

Under load, the state should move toward:

```text
LOAD -> MAX
```

If the Home Assistant-side template sees a telemetry mismatch, it should show:

```text
SUS
```

## Notes

Redline uses WTG-discovered Home Assistant entities. It does not create MQTT sensors directly.

WTG is the telemetry source. Redline is a dashboard interpretation layer.
