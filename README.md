# HASS-plate-recognizer
This is a fork of robmarkcole/HASS-plate-recognizer which utlizes the codeproject.ai local plate recognizer instead of ALPR at platerecognizer.com


Read vehicle license plates with codeproject.ai (get it from https://www.codeproject.com/)

This integration adds an image processing entity where the state of the entity is the number of license plates found in a processed image. Information about the vehicle which has the license plate is provided in the entity attributes, and includes the license plate number and confidence (in a scale 0 to 1) in this prediction. For each vehicle an `platerecognizer.vehicle_detected` event is fired, containing the same information just listed. Additionally, statistics about your account usage are given in the `Statistics` attribute, including the number of `calls_remaining` out of your 2500 monthly available.

**Note** this integration does NOT automatically process images, it is necessary to call the `image_processing.scan` service to trigger processing.

## Home Assistant setup
Place the `custom_components` folder in your configuration directory (or add its contents to an existing `custom_components` folder). Then configure as below:

```yaml
image_processing:
  - platform: platerecognizer_local
    api_token: your_token (For now, any value)
    regions: (Not used, just to not ruin the config flow until cleaned)
      - gb
      - ie
    watched_plates:
      - kbw46ba
      - kfab726
    save_file_folder: /config/images/platerecognizer/
    save_timestamped_file: True
    always_save_latest_file: True
    mmc: True (For now, any value)
    detection_rule: strict (For now, any value)
    region: strict (For now, any value)
    server: http://yoururl:8080/v1/plate-reader/ (URL To your local Codeproject site)

    source:
      - entity_id: camera.yours
```
Then, **restart** your Home Assistant

Configuration variables:
- **api_key**: Not needed.
- **regions**: Not needed.
- **watched_plates**: (Optional) A list of number plates to watch for, which will identify a plate even if a couple of digits are incorrect in the prediction (fuzzy matching). If configured this adds an attribute to the entity with a boolean for each watched plate to indicate if it is detected.
- **save_file_folder**: (Optional) The folder to save processed images to. Note that folder path should be added to [whitelist_external_dirs](https://www.home-assistant.io/docs/configuration/basic/)
- **save_timestamped_file**: (Optional, default `False`, requires `save_file_folder` to be configured) Save the processed image with the time of detection in the filename.
- **always_save_latest_file**: (Optional, default `False`, requires `save_file_folder` to be configured) Always save the last processed image, no matter there were detections or not.
- **mmc**: Not needed.
- **detection_rule**: Not needed.
- **region**: Not needed.
- **server**: (Required - if vision is enabled in codeproject, use http://**codeproject local url**:**port**/v1/vision/alpr)


- **source**: Must be a camera.

<p align="center">
<img src="https://github.com/robmarkcole/HASS-plate-recognizer/blob/main/docs/card.png" width="400">
</p>

<p align="center">
<img src="https://github.com/robmarkcole/HASS-plate-recognizer/blob/main/docs/main.png" width="800">
</p>

<p align="center">
<img src="https://github.com/robmarkcole/HASS-plate-recognizer/blob/main/docs/event.png" width="800">
</p>

## Making a sensor for individual plates
If you have configured `watched_plates` you can create a binary sensor for each watched plate, using a [template sensor](https://www.home-assistant.io/integrations/template/) as below, which is an example for plate `kbw46ba`:

```yaml
sensor:
  - platform: template
    sensors:
      plate_recognizer:
        friendly_name: "kbw46ba"
        value_template: "{{ state_attr('image_processing.platerecognizer_1', 'watched_plates').kbw46ba }}"
```

Depending on your license plate, you may recieve an template error due to variables not being able to start with a number. if so, here is another method to create the template sensor:
```yaml
sensor:
  - platform: template
    sensors:
      plate_recognizer:
        friendly_name: "kbw46ba"
        value_template: "{{ state_attr("image_processing.platerecognizer_1", "watched_plates")["kbw46ba"] }}"
```


## Video of usage
Checkout this excellent video of usage from [Everything Smart Home](https://www.youtube.com/channel/UCrVLgIniVg6jW38uVqDRIiQ)

[![](http://img.youtube.com/vi/t-XxCrdj_94/0.jpg)](http://www.youtube.com/watch?v=t-XxCrdj_94 "")
