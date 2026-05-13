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
5.  Update the `wifi_ssid` and `wifi_password` in the `substitutions` section. If you don't the ESP32-C3 will start a Fallback AP to connect to, where you can connect to your WIFI.
6.  Click **Install** > **Wirelessly**.

### ESPHome Code
*  **config.yaml** in the files

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
