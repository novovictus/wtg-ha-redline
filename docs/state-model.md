# State Model

Initial Redline state sequence:

```text
IDLE -> LOAD -> MAX -> LIMIT -> SUS
```

`SUS` is an override, not a higher load state.

## IDLE

Low driver-reported activity, low power, and normal temperature.

## LOAD

Work is happening, but the GPU is not near the top of its operating envelope.

## MAX

High activity and/or high memory-controller activity with power near the reported limit.

## LIMIT

Home Assistant-side display state for operating-envelope pressure, such as power near the reported cap or temperature near a configured ceiling.

## SUS

Telemetry mismatch override. Example: low GPU utilization, low power, low VRAM use, and pinned memory-controller utilization.
