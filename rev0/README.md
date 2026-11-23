# Rev 0: two-channel transmit pulser

Rev 0 is a 4-layer board that drives two transducers with bipolar high-voltage pulses. It was the first step: get the transmit side working and see echoes in water before building a receive chain.

**Status:** done. Built, brought up and tested in water.

<p align="center">
  <img src="../docs/images/rev0-board.jpg" width="560" alt="Rev 0 two-channel transmit pulser, powered on the bench">
</p>

## Board

On each channel an MD1213 gate driver switches a TC6320 complementary MOSFET pair between the +HV and -HV rails. The gate drive is AC-coupled through 10 nF / 200 V capacitors, so the ±5 V driver outputs can control MOSFETs whose sources sit on the high-voltage rails. Back-to-back ES1D diodes clamp each transmit node, and each channel leaves the board on an SMA connector.

An LM2776 charge pump makes the negative driver rail from the single +5 V input. The HV rails have bulk and ceramic decoupling and 1 MΩ bleed resistors so they discharge when the supplies are off. The pulse inputs (INA, INB), output enable and trigger lines come in on a 2×5 header with 10 kΩ pull-downs, so the outputs stay off at power-up. HV nets are routed in their own net class with 0.5 mm clearance and 0.5 mm tracks.

The switching stage was simulated in LTspice before layout. The board was designed in KiCad and ordered from JLCPCB.

## Specs

| Parameter | Value |
|---|---|
| Channels | 2 |
| Board | 4 layers, KiCad |
| HV rails | Designed for ±100 V, tested to ±60 V (limit of the bench supplies) |
| MOSFET pair | TC6320, 200 V |
| Gate drive | ±5 V, AC-coupled |
| Test pattern | 5 MHz, 60 ns per half-cycle with 40 ns dead time, 1 to 7 cycles, 1 kHz PRF |
| Measured frequency | 5.000 MHz (200 ns period) |

## Files

| Folder or file | Contents |
|---|---|
| [`PCB/`](PCB/) | KiCad project |
| [`PCB/rev0_Tx_Circuit.pdf`](PCB/rev0_Tx_Circuit.pdf) | Schematic |
| [`PCB/jlcpcb/production_files/`](PCB/jlcpcb/production_files/) | Gerbers, BOM and placement files sent for fabrication |
| [`testing/`](testing/) | Bring-up procedure, test report, LTspice model, DRC and ERC reports |
| [`datasheets/`](datasheets/) | Datasheets for the key parts |

## Results

The board was brought up in four stages, each with an acceptance criterion that had to pass before moving on. The full write-ups are in the [bring-up procedure](testing/Rev0_TX_Pulser_Bring-Up_Procedure.pdf) and the [test report](testing/Rev0_TX_Pulser_Test_Report.pdf).

<p align="center">
  <img src="../docs/images/rev0-bench-setup.jpg" width="49%" alt="Bench setup: supplies, breadboard distribution, AD2 and the board">
  <img src="../docs/images/rev0-water-test-setup.jpg" width="49%" alt="Water tank test setup">
  <br>
  <sub>Left: bench supplies, breadboard rail distribution, AD2 pattern generator and the board. Right: the full setup with the water tank.</sub>
</p>

1. **Unpowered checks.** Every reachable net checked for continuity and resistance against the layout. No solder faults, no rail-to-ground shorts.
2. **Power only.** HV rails walked up from 10 V with a 0.5 A current limit. The charge pump produced both driver rails, and with output enable held low both outputs stayed at 0 V at full rail voltage. Every later step depends on that interlock.
3. **First enable.** INA, INB and output enable from an AD2, verified on its logic analyzer before connecting. Clean bipolar switching into a 220 pF dummy load, with amplitude tracking the rail voltage.
4. **Water tank.** A second 5 MHz element used as an independent receiver. The received burst measures 200 ns per period (5.000 MHz). A target about 1 cm away returned an echo at 14 to 15 µs, consistent with the round trip at the speed of sound in water. Ring-down was 4 to 5 cycles with the transducer in air and close to none with its face in water.

<p align="center">
  <img src="../docs/images/scope-5mhz-period.jpg" width="49%" alt="Received burst with cursors one period apart, 200 ns, 5.000 MHz">
  <img src="../docs/images/scope-time-of-flight.jpg" width="49%" alt="Transmit feedthrough at t = 0 and the echo 29 microseconds later">
  <br>
  <sub>Left: received burst, cursors one period apart (200 ns, 5.000 MHz). Right: transmit feedthrough at t = 0 and the water-path echo 29 µs later.</sub>
</p>

**What didn't work, and why.** I could not resolve an interface below the front surface of a tissue target. The numbers rule out weak reflection and attenuation over a few millimetres. The real limits are on the receive side. The echoes are raw transducer output going straight into the scope, with no amplifier, filter or time-gain compensation, so millivolt returns sit near the noise floor. Transmit feedthrough and ring-down also blank the first few microseconds, which is exactly where a shallow interface returns. Raising the transmit voltage would scale the feedthrough along with the echo, so the next board is the receive chain rather than more drive.

**Open items**

- Switching times are not measured yet. The 100 MHz scope can't resolve edges faster than about 3.5 ns.
- Transducer capacitance is estimated at 220 pF and needs an impedance measurement.
- Dead time can't be safely reduced until a current-sense resistor on the HV rail can show shoot-through on the scope.
- The tests need a tissue-mimicking phantom with targets at known depths in place of hand-held targets.

## Test setup

- Tektronix TDS 2014 oscilloscope (100 MHz, 1 GS/s)
- Digilent Analog Discovery 2 as the pattern generator and logic analyzer
- Bench supplies, two 0 to 30 V units in series per HV rail plus a fixed 5 V unit
- 5 MHz thickness-gauge transducers with Lemo connectors, through SMA-to-Lemo adapters

## Getting the files

```bash
cd ~/Desktop
git clone https://github.com/peteryazdi/Ultrasound-Imaging-System.git
```

This works in Terminal on macOS and Linux and in PowerShell on Windows. Open `rev0/PCB/rev0_Tx_Circuit.kicad_pro` with KiCad 10.

## License

MIT. See [LICENSE](../LICENSE).
