# BOBOVR-B100-RE

Reverse engineering notes, measurements, and hardware experiments for the **BOBOVR B100** battery pack used with BOBOVR Quest head straps and charging accessories.

> [!WARNING]
> This project involves lithium-ion batteries and exposed battery contacts. The information here is experimental and may be incomplete or wrong. Do not short contacts, exceed the pack voltage range, or attempt high-current charging without proper current limiting and battery-safety precautions.

## Goals

* Document the B100 pogo-pin interface.
* Understand how the battery wakes from its low-power state.
* Identify the function/protocol of the remaining signal pins.
* Determine how BOBOVR fast charging through the pogo contacts works.
* Build an open test dock / breakout fixture.
* Explore compatible DIY battery and charger hardware.

## Pogo Layout

Orientation used throughout this repository:

```text
        Front

OO      OOO      OO
12      345      67

        Back
```

## Working Pinout

| Pin | Current identification         | Confidence                                          |
| --: | ------------------------------ | --------------------------------------------------- |
|   1 | V+                             | Confirmed                                           |
|   2 | V+                             | Confirmed                                           |
|   3 | Unknown, likely telemetry/data | Probable                                            |
|   4 | Unknown, likely telemetry/data | Probable                                            |
|   5 | Wake / presence detect         | Confirmed function, exact behavior still under test |
|   6 | GND                            | Confirmed                                           |
|   7 | GND                            | Confirmed                                           |

Pins 1 and 2 measure approximately **0.07 ohm** between each other, strongly indicating they are paralleled power contacts.

Pins 3 and 4 are not required for basic discharge. Insulating both contacts still allows the battery to power the head strap, although the strap reports **00% battery** and error code **01**.

Pin 5 is required for wake/presence detection. Insulating pins 3, 4, and 5 prevents the battery from waking.

## Wake Behavior

The B100 normally disables its main pogo output when removed from the head strap or charger.

A resistor from **pin 5 to GND** can wake the pack.

Observed results:

| Pin 5 to GND | Result        |
| -----------: | ------------- |
|      10 kOhm | Wakes         |
|      22 kOhm | Wakes         |
|      47 kOhm | Does not wake |

When successfully awakened during testing, the main output measured approximately **8.1 V** at the pack's then-current state of charge.

This suggests pin 5 is a low-power presence-detect input with a threshold rather than a simple direct ground contact.

## Pin 5 Standby Measurements

With the battery asleep, a very small apparent sensing current was observed on pin 5 using known resistors to ground.

| Resistance | Max measured voltage | Approx. inferred current |
| ---------: | -------------------: | -----------------------: |
|   100 kOhm |              14.4 mV |                 0.144 uA |
|    47 kOhm |               7.8 mV |                 0.166 uA |
|    22 kOhm |               3.9 mV |                 0.177 uA |

The close agreement suggests an always-on or periodically sampled detection current on the order of roughly **0.15 to 0.18 uA**.

Open-circuit pin 5 measurements in mV DC mode were observed to bounce roughly between:

```text
Min: 0.1 mV
Avg: 8.8 mV
Max: 33 mV
```

This may indicate a very weak bias or periodic polling/sensing behavior.

## Other Electrical Observations

* The B100 pogo output range is approximately **5.4 to 8.4 V**, consistent with a 2-series lithium-ion pack.
* The battery's USB-C charging input is rated **5 V / 3 A max**.
* The three middle pogo contacts are physically taller than the outer power contacts.
* Pins 3 and 4 appear electrically similar to each other and different from pin 5.
* Pins 3 and 4 are suspected to be telemetry/data, possibly I2C/SMBus or another two-wire protocol.
* Exact protocol behavior is not yet confirmed.
* After USB-C is disconnected, a residual voltage can remain briefly on the pogo output and decay. This is not assumed to be usable main-output power.

## Diode-Mode Observations

Preliminary diode-mode measurements suggest pins 3 and 4 behave similarly to each other.

Observed approximately:

```text
Pin 3 to GND:
~0.93 V one polarity
OL reverse polarity

Pin 4 to GND:
~0.93 V one polarity
OL reverse polarity
```

Pin 5 behaves differently, supporting the idea that it serves a different function than pins 3 and 4.

Observed approximately:

```text
Pin 5 to V+:
~0.72 V one polarity
OL reverse polarity

Pin 5 to GND:
~0.11 V in both polarities
```

These values should be treated as preliminary because internal semiconductor paths can make diode-mode measurements difficult to interpret.

## Current Working Theory

The current working model is:

```text
1 2      3    4      5      6 7
V+ V+   DATA DATA   WAKE    GND GND
```

Pins 3 and 4 likely provide battery telemetry or communication.

Pin 5 appears to be a low-power presence/wake input that the battery monitors even while its main discharge path is disabled.

The head strap does not need to be connected to the Quest for the B100 to wake, which implies the wake mechanism is powered by the B100 itself rather than by the headset.

## Current Roadmap

* [x] Identify power contacts.
* [x] Identify the wake-required contact.
* [x] Reproduce battery wake externally with a passive resistor.
* [ ] Characterize the exact pin-5 wake threshold.
* [ ] Build a pogo-pin breakout / test dock.
* [ ] Determine whether pogo V+/GND accepts charging current.
* [ ] Characterize pins 3 and 4.
* [ ] Determine whether pins 3 and 4 use I2C, SMBus, or another protocol.
* [ ] Determine how BOBOVR pogo fast charging works.
* [ ] Document safe DIY battery-interface hardware.
* [ ] Create open-source replacement battery / adapter concepts.

## Planned Test Dock

The first hardware target is a simple breakout fixture with all seven pogo contacts exposed.

Initial wiring:

```text
1 + 2 -> V+
6 + 7 -> GND
5 -> 10 kOhm -> GND

3 -> breakout only
4 -> breakout only
```

This should provide a repeatable way to wake the B100 and access all signal pins without needing to probe the narrow head-strap slot.

## Repository Structure

```text
docs/
  pinout.md
  measurements.md
  charging-research.md

hardware/
  test-dock/

captures/
  logic/
  photos/
```

## Status

This is active reverse-engineering work.

Treat all unconfirmed pin functions, communication theories, and charging behavior as provisional until independently reproduced.

Contributions, measurements, logic captures, teardown photos, and compatible hardware findings are welcome.
