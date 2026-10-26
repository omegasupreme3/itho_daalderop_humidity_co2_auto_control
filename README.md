# itho_daalderop_humidity_co2_auto_control
Let Home Assistant automatically control ventilation demand on your Itho Daalderop heat exchanger based on CO2/humidity sensors. It bypasses the built-in handling of the ventilation demand on the heat exchanger.

> [!IMPORTANT]
> The automation was generated using ChatGPT in multiple stages. **I did not review any of it**. Use the script 'as is' as I don't take any responsibility for it. You are on your own here. If you come across bugs or have feature requests, you can create an issue. However I don't do any support. So I might or might not look at it at all ...

> [!IMPORTANT]
> The automation contains an extensive explanation of how it should be configured and how it works. You should read that.


# General
The automation can be configured with:
- various thresholds
- how fast increase/decrease of ventilation demand should happen
- etc.

It also uses priorities in respective order high to low:
1. Manual (low, medium, high, timer10/20/30)
2. Humidity
3. CO2

Details:
- For humidity/CO2 it uses the automatic (night) mode
- The timers (10/20/30) will revert to the automatic mode (or automatic night mode if that was previously active)
- By default the automatic night mode has 10% _(configurable)_ higher ventilation demand than the normal automatic mode
- In case a CO2 sensor fails for 5 minutes or longer _(configurable)_, ventilation demand will go to 50% _(configurable)_

## My setup
I've been using this at home for some time now without any problems. My setup is:
  - Itho Daalderop HRU 350 ECO
  - 2x Itho Daalderop CO2 sensor (04-00045)
  - 1x Itho Daalderop Humidity sensor (04-00046)
  - 1x Sonoff SNZB-02P (as humidity sensor)
  - 4x Sonoff AirGuard CO2 SAWF-08P (as CO2 sensor)


# Assumptions
- It uses the WiFi add-on (https://github.com/arjenhiemstra/ithowifi). It assumes the add-on was properly setup, connected to and configured in Home Assistant (use the official integration IthoWiFi via HACS)
- MQTT is configured and operational on the WiFi add-on. The _MQTT base topic_ must be _ithohru_. If you have a different value, update the corresponding variable in the automation as well
- The automation can use sensors and remotes from different manufacturers as long as they are properly configured and working in Home Assistant

> [!CAUTION]
> If you do have sensors/remotes from Itho Daalderop, the automation assumes **all those sensors/remotes are joined with the WiFi add-on only**. This is because the automation takes over handling ventilation demand from the heat exchangers built-in handling. If a sensor/remote is still connected to the heat exchanger and changes the ventilation demand, **the automation will NOT be aware of that** and this may lead to unwanted behavior of the heat exchanger. All sensors/remotes include: CO2/humidity sensors, Spider, physical remotes to set timers/auto/low/med/high modes and so on. Recommended setup is to only have the virtual remote (see step 3 below) and CO2 RF device (see step 5 below) be paired with the heat exchanger.


# Quit setup
Below is a quick configuration for the WiFi add-on just to make the automation work. You should follow the official setup guides prior to that. The automation assumes the WiFi add-on is at least on firmware version 3.14.
1. In _System settings_ set the following (leave everything else on default):
   - _Virtual remote settings_ -> _Map API key "command" to virtual remote 0_ == on
   - _Enable/Disable I2C commands_ -> _Ventilation status (31DA)_ == on
   - _Enable/Disable I2C commands_ -> _Fan system status (31D9)_ == on
   - _Autodetect CC1101 RF module_ -> _Itho RF remote support_ == on
   - _RF CO2 control_ -> Control interface == RF CO2
2. Clear all paired sensors/remotes on the heat exchanger. Follow the official guide of the heat exchanger and the remote. You only have to unpair ONE sensor/remote as that will clear all paired sensors/remotes on the heat exchanger
3. In _Virtual remotes_ configure a virtual remote:
   - Set a unique ID
   - The type must be _RFT AUTO_
   - Press _Update_
   - Power cycle the heat exchanger following the official guide. When in pairing mode, press _Join_ on the virtual remote
4. In _RF devices_ configure the RF remotes (e.g. CO2/humidity sensors, Spider, physical remotes). This is only necessary if they are from Itho Daalderop:
   - Set the WiFi add-on in learn mode
   - Follow the official guide for your sensors/remotes to pair them with the WiFi add-on
   - CO2/humidity sensors should be configured with _Receive_ for _Remote function_. Select the appropriate _Remote model_
5. In _RF devices_ configure a CO2 sensor. This is the sensor that allows the automation to stepless change the ventilation demand on the heat exchanger:
   - Set _Remote function_ to _Send_
   - Set _Remote model_ to _RFT CO2_
   - Press _Update_
   - Power cycle the heat exchanger following the official guide. When in pairing mode, press _Join_ for this remote

# Installation
1. Use the File Editor in Home Assistant
2. Create a directory _packages_ in the directory _homeassistant_
3. Upload the automation in the _packages_ directory
4. Restart Home Assistant

## UI control
The automation creates a UI ventilation control in Home Assistant for your dashboard, it's called: _input_select.itho_control_mode_. Use that control in case you manually want to set the different modes:
- automatic (night)
- low
- medium
- high
- timer10/20/30

> [!CAUTION]
> Only use the UI control that comes with the automation. **You should not use any other UI control** that allows changing ventilation demand as those will bypass the automation and this may lead to undesired behavior of the heat exchanger.
