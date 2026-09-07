# BOBOVR B100 Pogo Pinout

This document tracks the currently known and suspected functions of the **7 pogo contacts** on the BOBOVR B100 battery pack.

All information is based on direct measurements and functional testing. Unconfirmed signal functions should be treated as provisional.

## Physical Layout

Orientation used throughout this repository:

```text
        Front

OO      OOO      OO
12      345      67

        Back
```

The three middle contacts are physically taller than the outer four contacts.

## Current Pinout

| Pin | Function                 | Status             |
| --: | ------------------------ | ------------------ |
|   1 | V+                       | Confirmed          |
|   2 | V+                       | Confirmed          |
|   3 | Unknown data / telemetry | Probable           |
|   4 | Unknown data / telemetry | Probable           |
|   5 | Wake / presence detect   | Confirmed function |
|   6 | GND                      | Confirmed          |
|   7 | GND                      | Confirmed          |

Current simplified model:

```text
1 2      3    4      5      6 7
V+ V+   DATA DATA   WAKE    GND GND
```

## Pins 1 and 2 — V+

Pins 1 and 2 appear to be paralleled positive battery-output contacts.

Measured resistance between pins 1 and 2:

```text
~0.07 ohm
```

This strongly suggests they are directly connected internally and duplicated to share current across multiple pogo contacts.

The B100 pogo output is approximately within the range:

```text
5.4 V to 8.4 V
```

This is consistent with a 2-series lithium-ion battery pack.

When the battery is asleep and disconnected from a supported accessory, the main output is normally disabled.

When successfully awakened during testing, approximately:

```text
8.1 V
```

was measured between V+ and GND at the battery's then-current state of charge.

## Pins 6 and 7 — GND

Pins 6 and 7 are the main ground contacts.

These appear to be paralleled in the same way as pins 1 and 2.

For most measurements in this repository, either pin 6 or pin 7 may be used as the ground reference unless otherwise noted.

## Pins 3 and 4 — Unknown Data / Telemetry

Pins 3 and 4 are currently suspected to form a communication or telemetry pair.

Possible protocols include:

* I2C
* SMBus
* proprietary two-wire communication

This has not yet been confirmed.

### Functional Test

Pins 3 and 4 were both electrically insulated before attaching the B100 to the E3 Pro head strap.

Result:

* B100 still woke successfully
* head strap still powered/charged the headset
* battery display reported `00%`

This proves pins 3 and 4 are **not required for basic power delivery**.

Their likely purpose is therefore battery telemetry, identification, state-of-charge reporting, or related communication.

### Diode-Mode Behavior

Pins 3 and 4 behave similarly in diode mode.

Observed approximately:

```text
Pin 3 -> GND
~0.93 V one polarity
OL reverse polarity

Pin 4 -> GND
~0.93 V one polarity
OL reverse polarity
```

Their matching electrical behavior supports the theory that they are a related signal pair.

## Pin 5 — Wake / Presence Detect

Pin 5 is required for the B100 to wake its main power output.

### Tape Test

With pins 3 and 4 insulated:

```text
Battery wakes normally
```

With pins 3, 4, and 5 insulated:

```text
Battery does not wake
```

This isolates pin 5 as the contact required for presence/wake detection.

### Passive Wake Test

The B100 can be awakened without the original head strap by connecting pin 5 to GND through an appropriate resistor.

Observed results:

| Resistance from pin 5 to GND | Result        |
| ---------------------------: | ------------- |
|                      10 kOhm | Wakes         |
|                      22 kOhm | Wakes         |
|                      47 kOhm | Does not wake |

A 10 kOhm resistor is currently the recommended value for experimental breakout hardware because it has been confirmed to wake the pack reliably.

Current basic wake circuit:

```text
Pin 5
  |
  10 kOhm
  |
 GND
```

### Standby Sense Current

While the battery is asleep, pin 5 appears to have a very small internal sensing current.

Measured using known resistors from pin 5 to GND:

| Resistance | Maximum measured voltage | Approx. inferred current |
| ---------: | -----------------------: | -----------------------: |
|   100 kOhm |                  14.4 mV |                 0.144 uA |
|    47 kOhm |                   7.8 mV |                 0.166 uA |
|    22 kOhm |                   3.9 mV |                 0.177 uA |

These measurements are reasonably consistent with a sensing current on the order of:

```text
~0.15 to 0.18 uA
```

Open-circuit measurements on pin 5 in mV DC mode were observed to vary approximately:

```text
Min: 0.1 mV
Avg: 8.8 mV
Max: 33 mV
```

This suggests the wake-detection circuit may use an ultra-low-power bias or periodic sensing mechanism.

### Diode-Mode Behavior

Pin 5 behaves differently from pins 3 and 4.

Observed approximately:

```text
Pin 5 -> V+
~0.72 V one polarity
OL reverse polarity
```

And:

```text
Pin 5 -> GND
~0.11 V in both polarities
```

These readings suggest pin 5 is connected to active semiconductor circuitry rather than being a simple resistor or direct ground contact.

## Likely Electrical Arrangement

The current working theory is:

```text
        BOBOVR B100

1 2      3    4      5      6 7
| |      |    |      |      | |
V+       DATA DATA   DETECT  GND
|                         |
+---- main power path ----+
```

The pack appears to maintain a very-low-power detection circuit while the main discharge path is disabled.

When pin 5 sees an appropriate load to GND, the battery enables the main V+ output.

Pins 3 and 4 then likely provide battery telemetry to the attached accessory.

## Current Confidence Levels

### Confirmed

* Pins 1 and 2 are V+
* Pins 6 and 7 are GND
* Pin 5 is required for wake/presence detection
* 10 kOhm from pin 5 to GND wakes the battery
* Pins 3 and 4 are not required for power delivery

### Probable

* Pins 3 and 4 are a matched telemetry/data pair
* Pin 5 is an analog or threshold-based presence-detect input
* The main battery output is switched by internal BMS/power-path circuitry

### Unknown

* Exact pin 3 function
* Exact pin 4 function
* Communication protocol
* Logic voltage
* Device addresses/registers
* Whether pins 3/4 are involved in fast charging
* Whether pogo V+/GND directly accept charge current
* Exact wake threshold range on pin 5

## Next Tests

Planned tests include:

* build a pogo-pin breakout/test dock
* measure pins 3 and 4 while the battery is awake
* determine whether pins 3 and 4 use I2C/SMBus
* determine the exact resistance threshold for pin 5
* test whether controlled current can flow into the B100 through the pogo power contacts
* characterize official fast-charge behavior if possible

## Safety

The B100 contains lithium-ion cells capable of supplying significant current.

Do not:

* short V+ to GND
* inject unknown voltages into pins 3, 4, or 5
* exceed 8.4 V on the battery power path
* attempt uncontrolled charging through the pogo contacts
* assume unconfirmed pin functions are safe

Use current limiting and verify polarity before connecting external power.
