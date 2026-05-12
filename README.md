# ESP32-C3 Growbox Sensor (VPD, Temp, Humidity, Pressure)

This project provides an easy-to-build, professional-grade environmental sensor for grow tents. It uses an **ESP32-C3 SuperMini** and an **AHT20+BMP280** combo sensor to calculate **Vapour Pressure Deficit (VPD)** locally on the device.

## Features
- **Real-time VPD Calculation:** Uses the Tetens formula with an adjustable leaf temperature offset.
- **Dynamic Configuration:** Change your sensor variant and leaf offset directly in the code or UI without full reflashes.
- **Stability:** Includes a 24-hour auto-reboot and a 15-minute WiFi watchdog to ensure 24/7 uptime.
- **Noob-Friendly:** Highly commented code and simple "plug-and-play" wiring.

---

## 🛠 Step 1: Identifying the Parts
*   **The Brain:** [ESP32-C3 SuperMini](https://google.com) (It's about the size of a postage stamp).
*   **The Eye:** [AHT20 + BMP280 combo sensor](https://google.com) (Usually a small purple or blue board).
*   **The Wires:** 4 Jumper wires (Female-to-Female) or a soldering iron.

---

## 🔌 Step 2: Wiring (The "Plug and Play" Method)
You don't need to solder if you use jumper wires, but soldering is more reliable for long-term use in humid grow tents. Match the labels on the sensor to the pins on the ESP32-C3:


| Sensor Pin | ESP32-C3 Pin | Function |
| :--- | :--- | :--- |
| **VCC** | **3V3** | 3.3V Power |
| **GND** | **GND** | Ground (-) |
| **SDA** | **GPIO 8** | Data Line |
| **SCL** | **GPIO 9** | Clock Line |

---

## 💻 Step 3: Flashing the Code
1.  Plug your ESP32-C3 into your computer using a **USB-C cable**.
2.  Open [web.esphome.io](https://esphome.io) in a Chrome or Edge browser.
3.  Click **Connect**, select your ESP32, and click **Install** to put the base firmware on the chip.
4.  Once connected to your network, go to your ESPHome dashboard and create a new device.
5.  Paste the **ESPHome Code** (provided below) into the editor.
6.  Click **Install** > **Wirelessly**.

### ESPHome Code
```yaml
substitutions:
  device_name: "growbox-sensor"
  friendly_name: "Growbox Sensor"
  # SENSOR VARIANT: Options are AHT10, AHT20, AHT21, or AHT30
  aht_variant: "AHT20" 
  update_interval: "60s"

esphome:
  name: "\${device_name}"
  friendly_name: "\${friendly_name}"

esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: esp-idf

logger:
api:
ota:
  - platform: esphome

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  reboot_timeout: 15min 

# Daily maintenance reboot
interval:
  - interval: 24h
    then:
      - button.press: restart_button

button:
  - platform: restart
    name: "\${friendly_name} Restart"
    id: restart_button

# Dynamic Leaf Offset for VPD calculation
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
*   **Avoid the "Hot Box":** Don't mount the sensor directly on top of the ESP32 chip. The chip's heat will bleed into the sensor and ruin your readings. Give it 2-5 inches of space.
*   **Airflow:** Hang the sensor at canopy height (where the plant leaves are). Keep it away from direct light if possible, as LEDs can heat the sensor's casing.
*   **Pro Tip (VPD Tuning):** If you notice your VPD looks "wrong," check your **Leaf Temp Offset** in your dashboard. 
    *   **Intense LED Lighting:** Leaves might be same as air (Offset 0).
    *   **High Airflow/Low Light:** Leaves might be much cooler (Offset -3).

---

## 🛠 Troubleshooting
*   **Reading says "0" or "NAN":** Your SDA and SCL wires might be swapped. Try swapping the wires on pins 8 and 9.
*   **WiFi Dropping:** Ensure your grow tent isn't acting as a Faraday cage (common with Mylar). Move the ESP32 closer to the tent's opening.
