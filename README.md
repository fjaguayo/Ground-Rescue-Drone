# Ground Rescue Drone

A compact, teleoperated ground robot built with ESP32 that streams live video over WiFi and is controlled via a PS4 gamepad. The project came out of a final year assignment at Universidad de Málaga (Industrial Electronics Engineering), with the goal of building something actually useful for rescue or reconnaissance — not just another LED-blink demo.

<img width="526" height="262" alt="image" src="https://github.com/user-attachments/assets/a80578e5-50cd-4648-bab3-5e423deacafb" />

## What it does

The robot moves using differential drive (two DC motors, one per side) and sends a live video feed from an ESP32-CAM mounted at the front. You control it from a PS4 controller over Bluetooth, and watch the stream from any browser on the same network. It fits in tight spaces — the whole thing is about 10 cm long and 80 mm in diameter — which was the original design constraint: something that could navigate rubble or collapsed structures where larger robots can't go.

The chassis is fully 3D-printed, so it's easy to modify, reinforce, or repaint for different environments.

## Hardware

The brain is split between two microcontrollers. The ESP32-CAM (OV2640 sensor) handles video capture and serves the web stream. The ESP32 DevKit handles motor control, encoder reading, and PS4 pairing over Bluetooth.

For movement, the robot uses two DC motors rated at 120 rpm nominal (12V), though they run at 7.4V in practice, which gives a slow and controllable speed — good for navigating debris without losing control. Each motor drives an 80mm rubber RC wheel through an L298N H-bridge driver. Power comes from a 9V NiMH rechargeable battery with a step-down regulator feeding 5V to the logic boards.

The chassis is printed in PLA/PETG on an Ender 3 v3, using around 300-400g of filament in total. Battery life sits at around 10-15 minutes of continuous movement with the camera streaming. Not great, but enough for a proof of concept.

## Software

Written in Arduino IDE using the ESP32 framework. Two separate programs, one per microcontroller.

**Motor control (ESP32)**

Reads the PS4 left stick for forward/backward and the right stick for steering. Wheel encoders feed into a PID loop running at 100Hz via a FreeRTOS ticker, which keeps both motors at a consistent speed even on uneven terrain.

```cpp
const float KP = 1.0;
const float KI = 0.05;
const float KD = 0.02;
```

Before flashing, replace the MAC address with your own controller's:

```cpp
PS4.begin("AC:FD:93:FC:71:22");
```

**Camera server (ESP32-CAM)**

Creates a WiFi access point (SSID: `ESP32-CAM-AP`, password: `12345678`) and serves a web interface at `http://192.168.4.1`. Supports JPEG streaming up to UXGA resolution and RGB565 mode for face detection. Latency is typically 200-500ms depending on signal quality.

## Cost breakdown

Total hardware cost per unit comes out to around €32.85, which compares well to similar platforms. The mBot costs €45 and has no camera or encoders. Basic 2WD kits are cheaper but have no wireless video or proper controller support.

| Component | Qty | Total (€) |
|---|---|---|
| ESP32-CAM OV2640 | 1 | 8.50 |
| ESP32 DevKit | 1 | 8.00 |
| L298N driver | 1 | 9.95 |
| DC motors 120rpm | 2 | 9.00 |
| RC wheels 80mm | 2 | 4.00 |
| Wheel encoders | 2 | 7.12 |
| 9V NiMH battery | 1 | 5.00 |
| PLA/PETG filament | 300g | 5.40 |
| Cables, LEDs, PCB, misc | — | 7.88 |
| **Total** | | **€64.85** |

## Getting started

Print the chassis parts (STL files in `/cad`) and wire everything up following the Fritzing schematic in `/schematics`. Flash `motor_control.ino` to the ESP32, updating the PS4 MAC address first, then flash `cam_server.ino` to the ESP32-CAM. Power on, connect to `ESP32-CAM-AP` on your phone or laptop, open `http://192.168.4.1`, pair the PS4 controller, and drive.

## What's next

The most obvious gap is battery life — a LiPo swap would help a lot. Beyond that, the platform is set up to support obstacle avoidance sensors, a proper mobile app instead of the browser interface, and eventually an autonomous navigation mode. The architecture was designed with that in mind from the start.

## License

Open for educational and non-commercial use. Built at Universidad de Málaga.
