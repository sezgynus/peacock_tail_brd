# Peacock Tail Controller (PTC)

[English](README.md) | [Türkçe](README-tr.md)

**Peacock Tail Controller is an open-source controller PCB for multi-material systems used in 3D printers.** The board combines an RP2040-Zero controller, two removable TMC2209 SilentStepStick-compatible stepper-driver positions, a 5 V buck regulator, and dedicated connections for the actuator and sensor functions commonly required by an MMU.

The current KiCad design is marked **revision 0.1**, dated **2026-05-06**.

> This repository is a hardware project under active development. This documentation describes the current schematic/PCB design and does not claim firmware support or hardware validation that is not present in the repository.

## Project Overview

PTC is intended to centralize the electronics of a two-motor multi-material mechanism:

- a **gear stepper** for filament transport;
- a **selector stepper** for material-path selection;
- a **servo** output;
- an **encoder** input;
- a **selector/endstop** input;
- an **extra toolhead/sensor** input;
- a dedicated **LED** connection.

The board uses an **RP2040-Zero** module as its controller and exposes the step/direction/enable, UART and diagnostic signals required by two TMC2209 driver modules.

```text
                         Peacock Tail Controller
                 ┌──────────────────────────────────┐
                 │                                  │
                 │          RP2040-ZERO             │
                 │                 │                │
                 │       ┌─────────┴─────────┐      │
                 │       │                   │      │
                 │   Gear TMC2209      Selector    │
                 │                     TMC2209      │
                 │       │                   │      │
                 └───────┼───────────────────┼──────┘
                         │                   │
                    Gear motor         Selector motor

                 Servo  Encoder  Endstop  Extra  LED
                   │       │        │       │     │
                   └───────┴────────┴───────┴─────┘

Input VIN ──► motor power
     │
     └──► LM2596S-ADJ buck ──► +5 V ──► board peripherals
```

## At a Glance

| Area | Current design |
|---|---|
| Project | Peacock Tail Controller / PEACOCK TAIL BRD |
| Hardware revision | 0.1 |
| Schematic date | 2026-05-06 |
| MCU module | RP2040-Zero |
| Stepper drivers | 2 × TMC2209 SilentStepStick-compatible modules |
| Stepper roles | Gear + Selector |
| Motor connectors | 2 × 4-pin |
| Power input | VIN + GND screw terminal |
| On-board conversion | LM2596S-ADJ adjustable buck stage |
| Local regulated rail | +5 V |
| Logic rail | +3.3 V |
| Servo connector | 3-pin |
| Encoder connector | 3-pin |
| Endstop connector | 3-pin |
| Extra/toolhead connector | 3-pin |
| LED connector | 3-pin |
| TMC configuration | STEP/DIR/EN + UART/DIAG-related routing |
| Mechanical mounting | 3 × M3 mounting holes |
| CAD | KiCad |
| 3D assembly | `Board.step` |
| Production package | `production/PEACOCK_TAIL_BRD_v0.1.zip` |

## Hardware Architecture

The design is divided into five functional blocks visible in the schematic:

```text
┌─────────────┐
│     MCU     │  RP2040-Zero
└──────┬──────┘
       │
       ├──────────────► Gear stepper driver
       │
       ├──────────────► Selector stepper driver
       │
       ├──────────────► Servo
       ├──────────────► Encoder
       ├──────────────► Endstop
       ├──────────────► Extra / toolhead input
       └──────────────► LED output

VIN
 │
 ├────────────────────► TMC2209 motor supply
 │
 └──► 5 V buck stage ─► +5 V peripherals / controller supply path
```

The PCB therefore acts as a carrier and interconnect board around modular controller and stepper-driver hardware rather than integrating the RP2040 and TMC2209 ICs directly.

## RP2040-Zero Controller

The current schematic uses **U1: RP2040-ZERO**.

Signals connected to the controller include:

- `G-EN`, `G-STEP`, `G-DIR`;
- `S-EN`, `S-STEP`, `S-DIR`;
- `G-UART`, `S-UART`;
- `G-DIAG`, `S-DIAG`;
- `G-INDEX`, `S-INDEX`;
- `SERVO`;
- `ENCODER`;
- `ENDSTOP`;
- `EXTRA`;
- `LED`.

The repository does not currently contain the firmware/configuration that assigns these signals to a particular Klipper or Marlin runtime configuration, so firmware pin mappings and software behavior are intentionally not inferred here.

## Stepper Driver Channels

Two TMC2209 SilentStepStick-compatible module positions are present.

### Gear Driver — U3

The gear channel uses:

- `G-STEP`;
- `G-DIR`;
- `G-EN`;
- `G-UART`;
- `G-DIAG`;
- `G-INDEX`.

Its motor phases are routed to the four-pin **J7** connector.

### Selector Driver — U4

The selector channel uses:

- `S-STEP`;
- `S-DIR`;
- `S-EN`;
- `S-UART`;
- `S-DIAG`;
- `S-INDEX`.

Its motor phases are routed to the four-pin **J8** connector.

Each driver position includes local bulk capacitance on VIN and supporting pull-up/resistor components around its control signals.

## TMC2209 UART Routing

The PCB contains four solder jumpers associated with the two TMC2209 modules:

- **JP7 / JP8** — gear-driver UART-related routing;
- **JP9 / JP10** — selector-driver UART-related routing.

In the current PCB/netlist:

- JP7 and JP9 are defined as normally open solder jumpers;
- JP8 and JP10 are defined as bridged/closed solder jumpers.

These jumpers form part of the TMC module's UART/control routing. Their configuration should be treated as a hardware option and verified against the intended firmware/TMC2209 module variant before changing the factory PCB state.

## Power Architecture

### Main Input

**J6** is the two-pin main power connector:

| J6 pin | Net |
|---|---|
| 1 | GND |
| 2 | VIN |

VIN feeds the stepper-driver motor-power domain and the on-board DC/DC stage.

The repository does not specify a validated operating-input range for the assembled board. The safe supply range must therefore be determined from the populated components and the intended build rather than assumed from the connector name.

### 5 V Buck Converter

The local 5 V supply is built around **U2: LM2596S-ADJ**.

The power stage includes:

- **L1:** 47 µH;
- **D1:** SS34 Schottky diode;
- **RV1:** 10 kΩ adjustment potentiometer;
- **R5:** 1 kΩ feedback resistor;
- input/output filtering and decoupling capacitors.

This is an adjustable regulator topology. The populated board's output should be set and verified before connecting 5 V peripherals.

### Supply Rails

The design contains three principal supply nets:

- `VIN` — incoming/motor supply;
- `+5V` — buck-regulated peripheral/controller rail;
- `+3V3` — logic/sensor rail.

## Peripheral Connectors

### J1 — Servo

| Pin | Signal |
|---|---|
| 1 | SERVO |
| 2 | GND |
| 3 | +5V |

### J2 — Encoder

| Pin | Signal |
|---|---|
| 1 | ENCODER |
| 2 | GND |
| 3 | +3V3 |

### J3 — Extra / Toolhead

| Pin | Signal |
|---|---|
| 1 | EXTRA |
| 2 | GND |
| 3 | +3V3 |

### J4 — Endstop

| Pin | Signal |
|---|---|
| 1 | ENDSTOP |
| 2 | GND |
| 3 | +3V3 |

### J5 — LED

| Pin | Signal |
|---|---|
| 1 | LED |
| 2 | GND |
| 3 | +5V |

### J7 / J8 — Stepper Motors

J7 is connected to the gear-driver motor outputs and J8 to the selector-driver motor outputs. Each connector carries the four TMC2209 motor-phase outputs.

Because coil order depends on the connected stepper motor and harness, verify phase pairing before applying motor power.

## Sensor and Diagnostic Paths

The board exposes more than basic STEP/DIR control.

Both TMC channels have dedicated:

- UART nets;
- DIAG nets;
- INDEX nets.

This gives the hardware design access to TMC2209 diagnostic/control features. Whether those signals are used for sensorless homing, stall detection, status monitoring, or another function is a firmware-level decision and is not defined by this repository alone.

The separate `ENDSTOP` and `EXTRA` inputs also allow conventional external sensor connections independently of the TMC diagnostic signals.

## LED Output

J5 provides:

```text
LED signal
GND
+5 V
```

The schematic establishes the electrical connector and MCU signal but does not define a firmware LED protocol. It should therefore not be assumed to be NeoPixel, PWM, or another specific protocol until the firmware using the board defines that behavior.

## Mechanical Design

The PCB contains **three M3 mounting holes**:

- H1;
- H2;
- H3.

The repository includes a complete exported assembly model:

```text
Board.step
```

and custom component STEP models under:

```text
peacock_tail_brd.pretty/packages3d/
```

These assets allow enclosure and mechanical-clearance checks without requiring the full KiCad project to be opened.

## KiCad Design Assets

The current project uses modern KiCad files:

- `peacock_tail_brd.kicad_sch` — schematic;
- `peacock_tail_brd.kicad_pcb` — PCB layout;
- `peacock_tail_brd.kicad_pro` — project settings.

The schematic reports:

```text
Generator version : KiCad 10.0
Project title      : PEACOCK TAIL BRD
Revision           : 0.1
Date               : 2026-05-06
```

Legacy `.sch`, `.pro`, `.lib` and `.dcm` files are also retained in the repository. For the current hardware description, the `.kicad_*` files are the authoritative design set documented here.

## Custom Libraries and 3D Models

`peacock_tail_brd.pretty/` contains project-specific footprints, including:

- TMC2209 SilentStepStick module footprint;
- RP2040/Seeeduino-style module footprint assets retained by the project;
- custom pin-header and terminal-block footprints.

The `packages3d/` directory includes models for TMC2209 modules, the controller module, JST connectors, jumpers and screw terminals.

Some library asset names reflect earlier design iterations. Component identity in this README follows the **current schematic instances**, not historical footprint/model filenames.

## Production Files

The repository includes:

```text
production/PEACOCK_TAIL_BRD_v0.1.zip
```

plus dated backup archives and `production/netlist.ipc`.

The existence of a production archive indicates that fabrication outputs have been generated for revision 0.1. It does **not**, by itself, establish that the revision has completed electrical validation, assembly validation, or long-term printer testing.

## Integration with 3D Printer Firmware

The hardware is structured for MMU-style motion/control and exposes the signals needed by external firmware.

The previous project description mentions Klipper and Marlin as intended integration targets, but this repository currently contains **PCB design assets only**. It does not contain:

- a Klipper MCU configuration;
- Klipper `printer.cfg` examples;
- Happy Hare configuration;
- Marlin board definitions;
- RP2040 firmware source;
- a tested software pin map.

For that reason, this README documents the hardware interface without claiming drop-in compatibility with a specific firmware configuration.

## Bring-Up Checklist

Before powering a newly assembled board:

1. inspect the PCB for solder bridges and assembly errors;
2. verify there is no short between VIN, +5 V, +3.3 V and GND;
3. power the regulator section without sensitive peripherals where practical;
4. adjust and verify the LM2596 output before attaching 5 V loads;
5. confirm the controller's 3.3 V rail;
6. verify TMC2209 module orientation before insertion;
7. confirm stepper coil pairing on J7/J8;
8. verify the intended solder-jumper configuration;
9. connect sensors/servo/LED only after checking their required voltage and signal convention;
10. begin motor testing with conservative driver current settings.

## Current Project Status

The repository labels the project as **work in progress**, and that remains the appropriate status for the available design artifacts.

Current repository evidence includes:

- revision 0.1 schematic and PCB;
- RP2040-Zero controller design;
- two TMC2209 driver channels;
- 5 V buck power stage;
- peripheral and sensor connectors;
- custom footprints and 3D models;
- complete board STEP export;
- revision 0.1 fabrication package.

What is not established by the repository is equally important: there is no committed firmware, validation report, assembly guide, BOM, electrical test record, or documented production qualification.

## Repository Structure

```text
peacock_tail_brd/
├── peacock_tail_brd.kicad_sch
├── peacock_tail_brd.kicad_pcb
├── peacock_tail_brd.kicad_pro
├── Board.step
├── A4_ISO5457-1999_ISO7200-2004-OpenEYE.kicad_wks
├── fabrication-toolkit-options.json
├── fp-lib-table
├── sym-lib-table
├── peacock_tail_brd.pretty/
│   ├── *.kicad_mod
│   └── packages3d/
│       └── *.step / *.STEP
├── production/
│   ├── PEACOCK_TAIL_BRD_v0.1.zip
│   ├── netlist.ipc
│   └── backups/
├── legacy KiCad files
└── README.md / README-tr.md
```

## License

The repository does not currently define a project license. Until a license is added, do not assume permissions beyond those granted by applicable copyright law.
