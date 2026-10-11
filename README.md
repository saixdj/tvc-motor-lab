# TVC Motor Lab

An interactive 3D simulator and design tool for a **finless, two-motor thrust-vector-controlled rocket that lands propulsively**. Built for Team Parallax's entry in the World Space Week 2026 Reusable Rocket Challenge.

**Live app:** https://saixdj.github.io/tvc-motor-lab/

The vehicle flies on a **GATI F-20** ascent motor and lands on a **GATI E-30** descent motor. Both motors sit on one two-axis gimbal at the tail ring, driven by two servos through pushrods. When the descent motor lights, its thrust bursts the coupler and pushes the spent ascent motor out. A LilyGO **T-Beam Supreme** (ESP32-S3) runs the flight software.

## What the app does

- **3D simulation** of the full flight: liftoff, ascent burn, coast, apogee, descent, coupler separation, landing burn and touchdown. It includes cutaway and exploded views, follow cameras, a timeline and live telemetry.
- **Landing solver**: works out the speed-based ignition trigger for a motor that can't be throttled. It shows the safe ignition corridor and runs a Monte Carlo over ignition delay, sensor error, motor impulse and mass.
- **Build tab**:
  - stability criteria for each flight phase
  - avionics placement drawn from a component mass table
  - the descent-igniter circuit and servo power wiring
  - generated flight code
  - a ballast study
- **Mission tab**: every input is editable. That includes the motors (simple specs, `.eng` thrust curve or BATES grain design), the component masses and positions, the body, legs and gimbal geometry, the drag device and the error budget.

## Firmware

`firmware/tvc_lander/tvc_lander.ino` is the flight sketch for the recommended setup: a ~170 mm nose flare plus 100 g of ballast at the tail ring. The **Build** tab regenerates it from whatever is in the Mission tab.

- **Board:** ESP32S3 Dev Module, USB CDC On Boot enabled, 8 MB flash, QSPI PSRAM.
- **Libraries:** copy them from the `lib` folder of [LilyGo-LoRa-Series](https://github.com/Xinyuan-LilyGO/LilyGo-LoRa-Series): SensorLib, XPowersLib, Adafruit_BME280, Adafruit_Sensor, Adafruit_BusIO and U8g2.
- **Pins:**
  - GPIO21: igniter MOSFET gate
  - GPIO3: continuity sense
  - GPIO38 and GPIO39: pitch and yaw servos
  - GPIO48: parachute hatch
  - GPIO46: flare release

**Bench test with the motors removed before any flight.** Hold BOOT just after power-up to enter test mode. Lean the rocket toward a servo: the nozzle's open end must swing away from the lean. If it doesn't, flip `SERVO_DIR_P` or `SERVO_DIR_Y`.

## Status and limits

The numbers come from a model, not from measurements:

- **Motor curves:** built from Thrust Tech's listed peak thrust, total impulse and burn time.
- **Motor masses, the drag coefficients and the component positions:** estimates.
- **Stability margins:** rules of thumb for a finless body.

**Testing so far:** the sketch compiles against stub headers that mirror the real library signatures, and in a software-in-the-loop run it matched the model's ignition time. It has not been compiled with the real ESP32 toolchain or run on hardware yet.

Before trusting a trigger height, measure:

1. the real ignition delay
2. the real motor and component masses
3. the balance point of the assembled rocket

## Running locally

It's a single static HTML file. Open `index.html` in a browser, or serve the folder:

```
python3 -m http.server
```

React, Three.js, Babel and Tailwind load from public CDNs.
