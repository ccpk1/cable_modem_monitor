T## How to Install the Cable Modem Signal Health Sensor

This is a Template Blueprint that dynamically creates a sensor which calculates a `Good`, `Fair`, or `Poor` health grade for your modem. It evaluates all downstream/upstream channels, SNR, and error rates against industry-standard and DOCSIS thresholds.

> **⚠️ Note on Template Blueprints:** > While Home Assistant fully supports Template Blueprints in the backend engine, the graphical "Import Blueprint" button in the UI does not support them yet. Installation requires a quick manual file copy and a few lines of YAML.

### Step 1: Install the Blueprint File

1. Using a file editor (like the Studio Code Server add-on or File editor), navigate to your Home Assistant `config` folder.
2. Create the following folder path if it does not already exist:
`config/blueprints/template/cable_modem_monitor/`
3. Download the `signal_health_sensor.yaml` file from this repository and place it in that folder.

### Step 2: Find Your Modem's Device ID

To tell the blueprint which sensors to grade, you need your modem's internal Home Assistant Device ID.

1. In Home Assistant, go to **Settings > Devices & Services > Devices > Cable Modem Monitor**.
2. Search for your modem (e.g., Arris S33, Motorola MB8611) and click on it.
3. Look at the URL in your web browser. Copy the long, 32-character alphanumeric string at the very end of the URL.
*(Example: `http://homeassistant.local:8123/config/devices/device/5d282d541a3a96afe436ef7f99a2cf79`)*

### Step 3: Add it to your Configuration

Open your `configuration.yaml` file and add the following block. Replace the `modem_device` string with the ID you copied in Step 2.

**Note:** You do *not* need to copy all the commented lines below into your configuration! You only need to include the `modem_device` line. The optional overrides are listed here simply so you know what is available to copy and tweak if you need to.

```yaml
template:
  - use_blueprint:
      path: cable_modem_monitor/signal_health_sensor.yaml
      input:
        # --- REQUIRED: Replace this string with your modem's HA Device ID ---
        modem_device: "PASTE_YOUR_DEVICE_ID_HERE"

        # ====================================================================
        # ALL INPUTS BELOW ARE OPTIONAL.
        # Uncomment and modify them ONLY if you need to override the defaults.
        # ====================================================================

        # --- Update Trigger Time Overrides ---
        # Controls evaluation frequency. Default is every 10 minutes AND on poll event.
        # TIP: Use a static minute like '0' and a static entity to force a rigid hourly loop.

        # update_time_interval: "/10"

        # --- Update Trigger Sensor Overrides ---
        # Target entity that changes on every single poll to force instant calculations.
        # If your modem lacks an uptime sensor, you can uncomment this and swap it out for:
        #  - sensor.cable_modem_tcp_latency
        #  - sensor.cable_modem_ds_ofdm_ch_33_corrected
        #  - sensor.cable_modem_model_name (To force a static Time-Only mode)

        # update_trigger_sensor: sensor.cable_modem_system_uptime

        # --- Entity Overrides ---
        # Explicitly targets specific integration sensors if names vary.

        # ds_channel_count: sensor.cable_modem_ds_channel_count
        # us_channel_count: sensor.cable_modem_us_channel_count
        # docsis_status: sensor.cable_modem_docsis_status
        # error_rate: sensor.cable_modem_rate_uncorrected_errors

        # --- Regex Overrides ---
        # Search expressions used to dynamically aggregate channel groups.

        # ds_power_regex: "cable_modem_ds_.*_power$"
        # ds_snr_regex: "cable_modem_ds_.*_snr$"
        # us_power_regex: "cable_modem_us_.*_power$"

        # --- SNR Threshold Overrides ---
        # Minimum signal-to-noise floor levels measured in dB.

        # snr_fair_min: 33.0
        # snr_poor_min: 25.0

        # --- Error Rate Threshold Overrides ---
        # Tolerable uncorrectable code word volume limits per minute.

        # error_rate_fair_max: 10.0
        # error_rate_poor_max: 50.0

        # --- Downstream Power Threshold Overrides ---
        # Tolerable absolute deviations from 0 dBmV, and channel spread delta boundaries.

        # ds_power_fair_max: 7.0
        # ds_power_poor_max: 15.0
        # ds_delta_fair_max: 3.0
        # ds_delta_poor_max: 4.0

        # --- Upstream Power Threshold Overrides ---
        # Return path transmission ranges and maximum saturation limit boundaries.

        # us_power_good_min: 35.0
        # us_power_good_max: 50.0
        # us_power_poor_min: 30.0
        # us_power_poor_max: 55.0


```

*(Note: The blueprint automatically discovers your channels and uses standard DOCSIS thresholds. See the blueprint file for all available input variables and descriptions).*

### Step 4: Restart your Home Assistant or Reload and Verify

1. Go to **Developer Tools > YAML**.
2. Click **Check Configuration** to ensure your spacing is correct.
3. Click the **Template Entities** button to reload your templates.
*(Note: If this is the very first time you have used the `template:` integration, that button will not appear yet. You will need to do one full Home Assistant Restart first).*
4. Navigate to the **States** tab and search for `sensor.cable_modem_signal_health`.

Your sensor is now active! It will automatically re-evaluate based on your triggers and surface the governing reason and exact channel data in its attributes.
