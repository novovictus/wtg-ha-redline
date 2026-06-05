# WTG HA Redline

Home Assistant dashboard and template examples for WTG MQTT telemetry.

WTG reports the GPU telemetry. Redline visualizes it.

## Start here

If WTG is already talking to Home Assistant and Home Assistant has already created the WTG GPU entities, start with the install guide:

- [Install Redline after WTG discovery is working](docs/install-redline-from-wtg-discovery.md)

The current Redline template package is here:

- [packages/wtg_gpu_redline.yaml](packages/wtg_gpu_redline.yaml)

The current basic dashboard card example is here:

- [dashboards/redline-basic.yaml](dashboards/redline-basic.yaml)

Before using the package or dashboard example, replace `YOUR_HOSTNAME` with the hostname portion from your discovered WTG entity IDs.

Example discovered entity:

```text
sensor.wtg_bench_gpu_0_gpu_0_power
```

In that example, replace `YOUR_HOSTNAME` with:

```text
bench
```

The hostname is the part after `wtg_` and before `_gpu_0_`.

Expected user flow:

```text
WTG MQTT discovery working -> find hostname from discovered entity IDs -> install host-scoped Redline template package -> add host-scoped Redline dashboard cards
```

## Recommended package install shortcut

The Home Assistant web-based terminal can be unreliable when pasting large YAML files. A local SSH session is usually cleaner because the package can be downloaded directly from GitHub and edited in the stream before it is written to Home Assistant.

From a local terminal, SSH into Home Assistant:

```sh
ssh root@homeassistant.local
```

If `homeassistant.local` does not resolve, use the Home Assistant IP address:

```sh
ssh root@<home_assistant_ip>
```

Create the packages directory:

```sh
mkdir -p /config/packages
```

Download the Redline package and replace `YOUR_HOSTNAME` in one step. Replace `<hostname>` with the hostname portion from your discovered WTG entity IDs:

```sh
curl -fsSL https://raw.githubusercontent.com/novovictus/wtg-ha-redline/main/packages/wtg_gpu_redline.yaml | sed 's/YOUR_HOSTNAME/<hostname>/g' > /config/packages/wtg_<hostname>_redline.yaml
```

Example for discovered entities beginning with `sensor.wtg_bench_`:

```sh
curl -fsSL https://raw.githubusercontent.com/novovictus/wtg-ha-redline/main/packages/wtg_gpu_redline.yaml | sed 's/YOUR_HOSTNAME/bench/g' > /config/packages/wtg_bench_redline.yaml
```

For a second WTG host, install a second package file with that host's name:

```sh
curl -fsSL https://raw.githubusercontent.com/novovictus/wtg-ha-redline/main/packages/wtg_gpu_redline.yaml | sed 's/YOUR_HOSTNAME/rog/g' > /config/packages/wtg_rog_redline.yaml
```

Verify the placeholder was replaced:

```sh
grep YOUR_HOSTNAME /config/packages/wtg_<hostname>_redline.yaml
```

No output means the placeholder is gone.

Then continue with the install guide to enable packages, check configuration, restart Home Assistant, and add the dashboard cards.

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

## Current WTG MQTT availability

The latest published WTG release is still the previous beta release line. WTG MQTT support is available from the WTG `main` branch after the MQTT work was merged, but it is not yet published as a packaged release artifact.

For now, use this repo as a dashboard/example workspace for source-built WTG MQTT testing.

The intended flow is:

```text
build WTG from main -> publish MQTT telemetry -> test Redline cards in Home Assistant -> publish the next demo-ready WTG release
```

Once the next WTG release is cut, this README can be updated to point users at the release artifact instead of source-built `main`.

## Expected MQTT telemetry shape

WTG publishes one JSON state payload per GPU to a topic like:

```text
wtg/<node_id>/gpu<index>/state
```

Example:

```text
wtg/bench1/gpu0/state
```

The current Redline examples are aligned with the WTG `main` MQTT state payload schema:

```json
{
  "wtg_version": "0.2.4",
  "payload_schema": 1,
  "tick_seq": 42,
  "tick_ts": "2026-06-04T12:34:56.789Z",
  "host": "WTG-BENCH",
  "node_id": "bench1",
  "gpu_index": 0,
  "gpu_name": "NVIDIA GeForce RTX 3060 Ti",
  "gpu_uuid": "GPU-example-uuid",
  "driver_version": "581.95",
  "cuda_driver_version": "13.0",
  "compute_mode": "Default",
  "perf_state": "P2",
  "pci_bus_id": "00000000:01:00.0",
  "temp_c": 54,
  "util_gpu_pct": 89,
  "util_mem_controller_pct": 95,
  "vram_used_mib": 5588,
  "vram_total_mib": 8192,
  "power_w": 199.4,
  "power_limit_w": 200.0
}
```

These fields are treated as the input signal for Home Assistant examples. The dashboards in this repo may add template sensors, derived display scores, or state labels, but those are Home Assistant-side conveniences only.

## WTG usage

This repo does not replace WTG and does not implement the MQTT publisher.

Use WTG to publish telemetry. Use this repo to experiment with Home Assistant display examples built from that telemetry.

For WTG source, release artifacts, and MQTT implementation details, see:

- <https://github.com/novovictus/WTG>
- <https://github.com/novovictus/WTG/releases>

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
Home Assistant-side display state for constraint evidence, such as temperature near a configured ceiling or future clock/power-envelope evidence. Near-power-cap operation by itself is treated as MAX, not LIMIT.

SUS
Telemetry mismatch override. Example: low GPU utilization, low power, low VRAM use, and pinned memory-controller utilization.
```

`SUS` should be treated as an override, not as a higher load state.

## Repo layout

```text
wtg-ha-redline/
  README.md
  LICENSE
  packages/
    wtg_gpu_redline.yaml
  dashboards/
    redline-basic.yaml
  examples/
    mqtt-payload-example.json
  docs/
    install-redline-from-wtg-discovery.md
    telemetry-boundary.md
    state-model.md
  assets/
    screenshots/
    mockups/
```

## Status

Early scaffold. MQTT publishing currently depends on a source build of WTG from `main`. Expect names, thresholds, and dashboard examples to change as the WTG MQTT/eGUI workflow settles.
