# WTG HA Redline

Home Assistant dashboard and template examples for WTG MQTT telemetry.

WTG reports the GPU telemetry. Redline visualizes it.

## What this is

WTG HA Redline is an example layer for turning WTG MQTT telemetry into Home Assistant dashboard cards. The first target is an arcade-style GPU Redline display with these Home Assistant-side states:

```text
IDLE -> LOAD -> MAX -> LIMIT -> SUS
```

These states are display interpretations for a dashboard. They are not WTG ground truth.

WTG remains the source-of-truth collector for driver-reported NVIDIA GPU telemetry. Home Assistant receives that telemetry over MQTT and can then build dashboards, gauges, warnings, and other local visualizations from those raw values.

## Boundary

WTG should publish driver-reported values such as utilization, memory-controller utilization, VRAM use, power, power limit, temperature, performance state, and other directly collected telemetry fields.

Redline can use those fields to create Home Assistant-side visual concepts such as:

- `IDLE`
- `LOAD`
- `MAX`
- `LIMIT`
- `SUS`
- redline-style gauges
- warning lamps
- dashboard badges

The Redline state model is intentionally separate from WTG. A Home Assistant template may classify a display state as `LIMIT` or `SUS`, but that does not mean WTG inferred that condition internally.

## Expected MQTT telemetry shape

The current examples assume that WTG publishes one JSON state payload per GPU to a topic like:

```text
wtg/<node_id>/gpu<index>/state
```

Example:

```text
wtg/bench1/gpu0/state
```

A representative payload may look like this:

```json
{
  "host": "WTG-BENCH",
  "gpu_index": 0,
  "gpu_name": "NVIDIA GeForce RTX 3060 Ti",
  "driver_version": "581.95",
  "util_gpu_pct": 89,
  "util_mem_controller_pct": 95,
  "vram_used_mib": 5588,
  "vram_total_mib": 8192,
  "power_w": 199.4,
  "power_limit_w": 200.0,
  "temp_c": 54,
  "perf_state": "P2"
}
```

These fields are treated as the input signal for Home Assistant examples. The dashboards in this repo may add template sensors, derived display scores, or state labels, but those are Home Assistant-side conveniences only.

## WTG usage

This repo does not replace WTG and does not implement the MQTT publisher.

Use WTG to publish telemetry. Use this repo to experiment with Home Assistant display examples built from that telemetry.

For WTG releases, source, and MQTT implementation details, see:

- <https://github.com/novovictus/WTG>
- <https://github.com/novovictus/WTG/releases/latest>

Detailed WTG configuration is intentionally not duplicated here. WTG MQTT and Home Assistant setup are expected to evolve with the WTG app and eGUI configurator.

## Initial state model

The first Redline card model is:

```text
IDLE -> LOAD -> MAX -> LIMIT -> SUS
```

Meaning:

```text
IDLE
Low driver-reported activity, low power, and normal temperature.

LOAD
Work is happening, but the GPU is not near the top of its operating envelope.

MAX
High activity and/or high memory-controller activity with power near the reported limit.

LIMIT
Home Assistant-side display state for operating-envelope pressure, such as power near the reported cap or temperature near a configured ceiling.

SUS
Telemetry mismatch override. Example: low GPU utilization, low power, low VRAM use, and pinned memory-controller utilization.
```

`SUS` should be treated as an override, not as a higher load state.

## Planned repo layout

```text
wtg-ha-redline/
  README.md
  LICENSE
  packages/
    wtg_gpu_redline.yaml
  dashboards/
    redline-basic.yaml
    redline-mushroom.yaml
    redline-button-card.yaml
  examples/
    mqtt-payload-example.json
    bench1-example.yaml
  docs/
    telemetry-boundary.md
    state-model.md
    thresholds.md
    screenshots.md
  assets/
    screenshots/
    mockups/
```

## Status

Early scaffold. Expect names, thresholds, and dashboard examples to change as the WTG MQTT/eGUI workflow settles.
