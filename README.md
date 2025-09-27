# hass-stuff

Assorted blueprints and configuration helpers for Home Assistant.

## TRV Calibrator Blueprint

The `blueprints/trv_calibrator.yaml` blueprint keeps a smart TRV calibrated against an external temperature sensor. It started life as [rotilho’s TRV Calibrator blueprint](https://github.com/rotilho/home-assistant-blueprints/blob/main/trv_calibrator.yaml) (MIT License) and now includes extended timing controls and retry safeguards to suit this setup.

### Install

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fstreaky%2Fhass-stuff%2Frefs%2Fheads%2Fmain%2Fblueprints%2Ftrv_calibrator.yaml)

### Inputs

- **Smart TRV** (`climate` entity)
- **Temperature Sensor** (`sensor` with `temperature` device class)
- **TRV Calibration Entity** (`number` entity exposed by the TRV)
- **Delay in minutes** – Wait after a trigger to let the TRV settle before calculating a new offset.
- **Minimum minutes between calibrations** – Enforces a minimum cool-down between calibration writes by checking the last time the calibration entity changed.
- **Update sync window (seconds)** – Requires the TRV’s temperature reading and the external sensor to have refreshed within this window of one another *and* of “now” before a calibration may proceed.
- **Sync wait timeout (minutes)** – Total time the automation will spend retrying to align the TRV and sensor updates once the minimum interval has elapsed (capped at one hour). Set to `0` to request an indefinite wait, still subject to the one-hour maximum.

### Behaviour

After a trigger the automation waits for the configured delay, then ensures the minimum interval since the last calibration has elapsed. Once the cooldown is over it repeatedly checks for the TRV and sensor states to be:

1. Updated within the sync window of each other, and
2. No older than the same window relative to “now”.

The automation keeps retrying until both checks pass or the earlier of the sync timeout or a one-hour safety cap is reached. If alignment never happens in that window, the run ends without writing a calibration. Otherwise, a new calibration value is computed (still bounded by the device’s min/max/step). If no meaningful adjustment is required, the existing calibration value is left untouched and the cool-down timer continues from the last actual update.
