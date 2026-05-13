# ESP32-C3 Growbox Sensor (Temp, Humidity, VPD, Pressure)

This project provides an easy-to-build environmental sensor for grow tents and greenhouses. It uses an **ESP32-C3 SuperMini** and an **AHT10** variant (AHT10, AHT20, AHT30, or AHT20+BMP280 combo) to calculate **Vapour Pressure Deficit (VPD)** locally on the device.

## 🌟 Features
- **Real-time VPD Calculation:** Uses the Tetens formula with an adjustable leaf temperature offset.
- **Dynamic Configuration:** Change the leaf temperature offset via the web UI or Home Assistant without reflashing.
- **Stability:** Includes a 24-hour auto-reboot and a 15-minute WiFi watchdog for 24/7 uptime.
- **Fallback Hotspot:** If WiFi fails, the device creates its own setup network for easy recovery.

---

## 🛠 Step 1: The Parts
- **Microcontroller:** ESP32-C3 SuperMini
- **Sensor:** AHT10/20/30 (optionally with BMP280 for air pressure)
- **Wires:** 4 wires
- **Enclosure:** some kind of enclosure. which lets air flow over and ideally has a separate compartment for the sensor. :)

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

To flash the YAML config you need the **ESPHome Device Builder** — a dashboard that compiles the YAML into firmware and flashes it to your device. Choose the option that fits your setup:

---

### Option A: Home Assistant Add-on (recommended)

1. In Home Assistant go to **Settings → Add-ons → Add-on Store**, search for **ESPHome** and click **Install**.
2. Start the add-on and click **Open Web UI**.
3. Click **+ New Device**, give it a name, select **ESP32-C3** as the board, and finish the wizard. It will generate a starter YAML — ignore it for now.
4. Click **Edit** on your new device and replace the entire contents with `config.yaml` from this project.
5. Plug your ESP32-C3 into the computer running Home Assistant via USB-C.
6. Click **Install → Plug into this computer** and wait for the compile and flash to finish (first compile takes a few minutes).

---

### Option B: No Home Assistant (standalone ESPHome)

Install ESPHome on your computer via Docker or pip, then use its web dashboard the same way as Option A. The official guide covers both methods:

👉 [ESPHome Getting Started — standalone install](https://esphome.io/guides/getting_started_command_line)

Once the dashboard is running, follow steps 3–6 from Option A above.

---

> **No WiFi credentials are needed in the code.** On first boot the device starts a setup hotspot automatically — see Step 4.

---

## 📶 Step 4: WiFi Setup

On first boot (or any time your saved WiFi network is unreachable), the device will start a fallback hotspot after about 60 seconds:

- **Network name:** `GrowSensor Setup`
- **Password:** `SetupPassword123`

Connect to that network with your phone or laptop. A captive portal should open automatically — if it doesn't, navigate to **http://192.168.4.1** in your browser. Enter your WiFi credentials and save. The device will connect and the hotspot will disappear.

Once on your network, the web UI is accessible at **http://growbox-sensor-01.local** (or the IP shown in your router's device list).

---

## 🏠 Step 5: Adding to Home Assistant

Home Assistant will detect the sensor automatically on your network — no API key or password is required.

1. In Home Assistant go to **Settings → Devices & Services**.
2. You should see a notification: *"New device discovered: Growbox Sensor 01"*. Click **Configure** and then **Submit**.
3. The sensor will appear as a new device with entities for Temperature, Humidity, VPD, Pressure (if BMP280 is connected), WiFi Signal, and Uptime.

If it is not discovered automatically, click **+ Add Integration**, search for **ESPHome**, and enter the device's IP address manually.

---

## 📍 Step 6: Positioning & Pro-Tips

- **Heat Management:** Keep the sensor 2–5 inches away from the ESP32 chip. The microcontroller generates heat that will skew your humidity and VPD readings if they are too close.
- **Placement:** Hang the sensor at canopy height, out of direct LED light to prevent the casing from heating up.
- **VPD Tuning:** Adjust the **Leaf Temp Offset** in your Home Assistant dashboard or the web UI based on your lighting:
  - **LEDs:** Leaves are often cooler than air (offset −2.0 to 0).
  - **HPS:** Leaves can be warmer than air (offset +1.0 to +3.0).

---

## 🔒 Optional: Adding Security

This config ships without an API encryption key or OTA password, which is fine for a sensor on a trusted home network. If you want to lock it down — for example if your network has many users or you just prefer it — you can add both in a few lines.

In your `config.yaml`, update the `api:` and `ota:` blocks:

```yaml
api:
  reboot_timeout: 0s
  encryption:
    key: "<your key here>"

ota:
  - platform: esphome
    password: "<your password here>"
```

To generate an API encryption key: open your ESPHome dashboard, click **Edit** on any device, and ESPHome can generate one for you — or use any base64-encoded 32-byte string. After saving and reflashing, Home Assistant will ask you to re-add the device and enter the key once.

---

## 🛠 Troubleshooting

- **"NAN" or 0 readings:** Usually means the I2C wires (SDA/SCL) are swapped. Try switching GPIO 8 and GPIO 9.
- **WiFi drops:** Grow tents with Mylar lining act as Faraday cages. If signal is weak, move the ESP32 closer to a tent opening or vent. The device will fall back to the setup hotspot automatically if the connection is lost for more than 60 seconds.
- **Not showing up in Home Assistant:** Try adding it manually via **Settings → Devices & Services → + Add Integration → ESPHome** and enter the device IP address.
- **Want to use a BMP280 pressure sensor?** Uncomment the `bmp280_i2c` block in `config.yaml`, adjust the I2C address if needed (`0x76` or `0x77`), and reflash.
