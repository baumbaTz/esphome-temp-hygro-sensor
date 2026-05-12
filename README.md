# ESP32-C3 Growbox Sensor (VPD, Temp, Humidity, Pressure)

This project provides an easy-to-build, environmental sensor for grow tents and greenhouses. It uses an **ESP32-C3 SuperMini** and an **AHT10** variant (AHT10, AHT20, AHT30, or AHT20+BMP280 combo) to calculate **Vapour Pressure Deficit (VPD)** locally on the device.

## 🌟 Features
- **Real-time VPD Calculation:** Uses the Tetens formula with an adjustable leaf temperature offset.
- **Dynamic Configuration:** Change sensor variants or leaf offsets via the UI without reflashing.
- **Stability:** Includes a 24-hour auto-reboot and a 15-minute WiFi watchdog for 24/7 uptime.
- **Fallback Hotspot:** If WiFi fails, the device creates its own "Setup" network for easy recovery.

---

## 🛠 Step 1: The Parts
*   **Microcontroller:** ESP32-C3 SuperMini
*   **Sensor:** AHT10/20/30 (Optionally with BMP280 for pressure)
*   **Wires:** 4 wires

---

## 🔌 Step 2: Wiring


| Sensor Pin | ESP32-C3 Pin | Function |
| :--- | :--- | :--- |
| **VCC** | **3V3** | 3.3V Power |
| **GND** | **GND** | Ground (-) |
| **SDA** | **GPIO 8** | Data Line |
| **SCL** | **GPIO 9** | Clock Line |

---

## 💻 Step 3: Flashing the Code
1.  Plug your ESP32-C3 into your computer via **USB-C**.
2.  Open [web.esphome.io](https://esphome.io) in Chrome or Edge.
3.  Click **Connect**, select your ESP32, and click **Install** for the base firmware.
4.  In your ESPHome dashboard, create a new device and paste the code below.
5.  Update the `wifi_ssid` and `wifi_password` in the `substitutions` section.
6.  Click **Install** > **Wirelessly**.

### ESPHome Code
```yaml
substitutions:
  device_name: "growbox-sensor-01"
  friendly_name: "Growbox Sensor 01"
  
  # --- USER CONFIGURATION START ---
  wifi_ssid: "Your_WiFi_Name"
  wifi_password: "Your_WiFi_Password"
  
  ap_ssid: "GrowSensor Setup"
  ap_password: "SetupPassword123"

  # Options: AHT10, AHT20, AHT21, or AHT30
  aht_variant: "AHT20" 
  update_interval: "60s"
  # --- USER CONFIGURATION END ---

esphome:
  name: "\${device_name}"
  friendly_name: "\${friendly_name}"

esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: esp-idf

captive_portal:
logger:
api:
ota:
  - platform: esphome

wifi:
  ssid: "\${wifi_ssid}"
  password: "\${wifi_password}"
  fast_connect: true
  reboot_timeout: 15min 
  ap:
    ssid: "\${ap_ssid}"
    password: "\${ap_password}"

interval:
  - interval: 24h
    then:
      - button.press: restart_button

button:
  - platform: restart
    name: "\${friendly_name} Restart"
    id: restart_button

number:
  - platform: template
    name: "\${friendly_name} Leaf Temp Offset"
    id: leaf_offset
    min_value: -10
    max_value: 5
    step: 0.1
    initial_value: -2.0
    optimistic: true
    restore_value: true
    unit_of_measurement: "°C"
    mode: box

i2c:
  sda: GPIO8
  scl: GPIO9
  scan: true
  id: bus_a

sensor:
  - platform: aht10
    variant: \${aht_variant}
    i2c_id: bus_a
    temperature:
      name: "\${friendly_name} Temperature"
      id: temperature_sensor
    humidity:
      name: "\${friendly_name} Humidity"
      id: humidity_sensor
    update_interval: \${update_interval}

  # This section is for combo sensors. If not using BMP280, you can delete this.
  - platform: bmp280_i2c
    i2c_id: bus_a
    address: 0x76
    pressure:
      name: "\${friendly_name} Pressure"
      id: pressure_sensor
    update_interval: \${update_interval}

  - platform: template
    name: "\${friendly_name} VPD"
    id: vpd_sensor
    unit_of_measurement: "kPa"
    accuracy_decimals: 2
    icon: "mdi:water-percent"
    update_interval: \${update_interval}
    lambda: |-
      if (id(temperature_sensor).has_state() && id(humidity_sensor).has_state()) {
        float t_air  = id(temperature_sensor).state;
        float t_leaf = t_air + id(leaf_offset).state; 
        float rh     = id(humidity_sensor).state / 100.0;
        // Tetens Formula
        float svp_air  = 0.6108 * exp(17.27 * t_air  / (t_air  + 237.3));
        float svp_leaf = 0.6108 * exp(17.27 * t_leaf / (t_leaf + 237.3));
        return svp_leaf - (rh * svp_air);
      }
      return NAN;

  - platform: wifi_signal
    name: "\${friendly_name} WiFi Signal"
  - platform: uptime
    name: "\${friendly_name} Uptime"
```

---

## 📍 Step 4: Positioning & Pro-Tips
*   **Heat Management:** Keep the sensor 2-5 inches away from the ESP32 chip. The microcontroller generates heat that will ruin your humidity/VPD readings if they are too close.
*   **Placement:** Hang the sensor at canopy height. Keep it out of direct LED light if possible to prevent the sensor casing from heating up.
*   **VPD Tuning:** Adjust the **Leaf Temp Offset** in your dashboard based on your lighting environment.
    *   **LEDs:** Leaves are often cooler than air (Offset -2.0 to 0).
    *   **HPS:** Leaves can be warmer than air (Offset +1.0 to +3.0).

---

## 🛠 Troubleshooting
*   **"NAN" or 0 Readings:** Usually means the I2C wires (SDA/SCL) are swapped. Try switching pins 8 and 9.
*   **Log Errors:** If you aren't using a BMP280 pressure sensor, you may see errors in the logs. You can safely ignore them or delete the `bmp280_i2c` block from the code.
*   **WiFi Drops:** Grow tents with Mylar lining act as Faraday cages. If the signal is weak, move the ESP32 closer to a tent opening or vent.
