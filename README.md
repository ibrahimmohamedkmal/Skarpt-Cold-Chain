# Skärpt Cold Chain HUB (SK-V-094)

THS202-compatible 1-Wire temperature and humidity hub with Bluetooth Low Energy broadcast and a relay output that controls the AC load in refrigerated trucks. Built for cold-chain monitoring with a Teltonika FMC130 tracker.

To an existing 1-Wire master the hub appears exactly like a THS202 sensor, so the controller needs no changes. The same readings are also advertised over Bluetooth Low Energy. The hub also monitors a door contact, sounds a local alarm when limits are exceeded, switches the truck's AC load on command from the FMC130, and reports its own health on a dedicated output.

> **Documentation-only repository.** Firmware source code and firmware binaries are proprietary and are not distributed here.

## Key features

- Drop-in THS202-compatible 1-Wire slave
- Bluetooth Low Energy broadcast every 100 ms, no pairing required
- DHT22 temperature and humidity, refreshed every 2.5 s (0.1 °C / 0.1 % RH resolution)
- **AC relay output (up to 15 A)** that switches the truck's AC load and follows the FMC130 command in real time
- Local audible alarm with distinct patterns for temperature, humidity and combined conditions
- Interrupt-driven door monitoring with status output and on-screen indication
- Health validation output, dual watchdogs and a fail-safe design: a failed sensor or silent bus is flagged instead of publishing bad data

## How it works

### Dual-core architecture

The hub runs on a dual-core ESP32, and the work is split deliberately between the two cores:

- **Core 1 runs only the 1-Wire engine.** Nothing else ever runs there, so the bus timing, which is measured in microseconds, can never be interrupted by sensor reads, display updates or Bluetooth activity.
- **Core 0 runs everything else:** the Bluetooth stack, DHT22 sensing, the LCD, the buzzer, the door input, the AC relay, the health output and the debug log.

Time-critical bus routines run from internal RAM, so flash or Bluetooth activity cannot stall them. Core 1 never waits on slow operations: it hands diagnostic events to core 0 through a small fixed-size queue and carries on.

### Interrupt-driven door monitoring

The door contact is handled by a hardware interrupt, not by polling. A door opening or closing is detected the moment it happens, even while the hub is busy reading the sensor or updating the display, and the door-status output and the on-screen indication update immediately.

### AC relay control

The hub contains a relay that switches the AC load in the truck, rated up to 15 A. The FMC130 commands it from its digital output (DOUT 2), and the hub mirrors that command to the relay in real time (approx. 5 ms). This lets the tracker platform turn the truck's AC load on and off remotely or by rule, for example based on the temperature the hub itself reports. The relay starts in the OFF state at power-up.

### Reliability and safety

- **Dual watchdogs:** a 5 s task watchdog runs on both cores. If either core stalls, the hub restarts itself automatically.
- **Memory safety:** the firmware was checked for memory leaks during development, and the real-time paths use fixed-size buffers and queues rather than dynamic allocation, so it is designed to run continuously for long periods in the field.
- **Health validation output:** goes LOW after 5 consecutive failed sensor reads, or if no valid bus activity is seen for 30 s. The last good reading is held during a sensor fault, so the controller never receives made-up data.
- **Self-consistent data:** every 1-Wire read returns a complete, consistent snapshot of the latest measurement.

## Specifications

| | |
|---|---|
| Power supply | 10–30 V DC (max 1.0 A) |
| AC load relay | Max 15 A |
| Sensing | DHT22: −40 to +80 °C, 0–100 % RH, ±0.5 °C / ±2 % RH typical |
| Bus | Dallas 1-Wire slave, external pull-up |
| Wireless | Bluetooth Low Energy, broadcast-only, 2.4 GHz |
| Processor | Dual-core ESP32, 240 MHz |
| Local display | 16×2 character LCD |
| Operating conditions | 0 to +45 °C, 10–90 % RH non-condensing |

See the [datasheet](docs/SK-V-094_Datasheet.pdf) for full specifications.

## Wiring to Teltonika FMC130

| Hub terminal | Connects to |
|---|---|
| DS-OUT | FMC130 pin 3, DIN 3 |
| VALID | FMC130 pin 4, DIN 2 |
| RLY-IN | FMC130 pin 9, DOUT 2 |
| O-WIRE | FMC130 pin 11, 1-Wire data |
| DS-IN | Wired door sensor, contact 1 (contact 2 to GND) |
| DHT-D / +3.3 / −GND | DHT22 DATA / VCC / GND |
| PWR IN + / − | 10–30 V DC supply |
| AC CON. 1 / 2 | AC load terminal A / AC supply L (load terminal B to N) |

![Wiring diagram](docs/SK-V-094_Wiring_Diagram.png)

Full diagram: [PDF](docs/SK-V-094_Wiring_Diagram.pdf)

## Documentation

- [Datasheet (PDF)](docs/SK-V-094_Datasheet.pdf)
- [Wiring diagram (PDF)](docs/SK-V-094_Wiring_Diagram.pdf)

## Contact

ibrahim.mohamed.kmal@gmail.com
## License

All rights reserved. © Skärpt Contracting Co. No part of this repository may be copied or redistributed without permission.
