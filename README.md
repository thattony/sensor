# ECE 437 Labs

Interactive study pages for ECE 437, one tab per lab:

- **I²C (Midterm):** bring up the LSM303DLHC accelerometer on I²C bus 1 and prove its readings are real, using gravity as the referee (Part 1), then measure the capacitance of both I²C buses from their rise times (Part 2).
- **Motor (Lab 8):** drive a DC gearmotor from the FPGA, and use the accelerometer on the arm as a protractor to measure where it really went.
- **Camera (Lab 9):** power up the CMV300 image sensor, set it over SPI, request frames, and prove every number with counters inside the FPGA.
- **Takeaways:** generic lessons from the labs that apply to most embedded and FPGA work.

**Live page: [thattony.github.io/sensor](https://thattony.github.io/sensor/)**

It's a single `index.html` file with no build step and no dependencies. Open it in a browser, or serve the folder:

```bash
python3 -m http.server 8771
```

## I²C

- **Hardware:** which bus and pins the sensor sits on, and its I²C addresses
- **I²C changes:** single write, single read and burst-read transactions, and the auto-increment bit
- **Registers:** CTRL_REG1_A and CTRL_REG4_A, with bits you can flip
- **Conversion:** how two bytes become acceleration in g
- **Known answer:** known-answer test
- **Stream axes:** configure, then burst read, convert and print in a loop
- **Gravity check:** the six-orientation gravity check
- **100 samples/s:** the timing budget for 100 samples/s, and using STATUS_REG_A to prove each sample is new
- **Two buses:** bus 0 carries three devices, bus 1 two, on the same 10 kΩ pull-ups
- **The rising edge:** open-drain lines, the clickable charge path, and t_r = ln 9 · RC
- **Inside the FPGA:** Python → FrontPanel → I²C state machine → open-drain pins, the four-tick bit, and the marker path to the scope
- **Burst FSM:** a step-through of the state machine for a 6-byte burst read
- **PMOD marker:** the 12-pin PMOD header and the one-line marker logic
- **Same edge:** where the PMOD marker fires (the R/W bit of 0x33) and the ACK edges it captures
- **100 edges:** 100+ edges per bus, histograms on common axes
- **Bus speed:** how bus capacitance sets the fastest I²C speed, with a C slider

## Motor

- **Pulses:** a pulse sets drive time, not an angle
- **Rig and home:** 300° of travel between hard stops, and why home goes at the center
- **Tilt:** how tilting the shaft turns gravity into an absolute angle
- **Stopped or moving:** read the angle only when stopped; log motion while moving
- **Driver:** EN1/DIR1 on the Pmod DHB1, and disabling before a change of direction
- **Pulse generator:** an FPGA counter that fires N pulses on EN1
- **Stall protection:** the accelerometer checks the arm keeps moving as told
- **Voltage and torque:** back-EMF, and how supply voltage sets speed and torque
- **Hysteresis:** gearbox backlash makes the stop point depend on direction

## Camera

- **Three paths:** SPI, the control pins and the video bus
- **One frame:** exposure, FOT and readout, to scale
- **SPI:** the sensor as a register file behind a 16-bit SPI frame
- **Known answer:** values split across two registers, low byte first
- **Power-up:** the sequencer's order
- **Clock and PLL:** the PLL band register and the CLK_OUT meter
- **FRAME_REQ:** a command with no reply, checked by what follows
- **Two regimes:** one frame at a time vs pipelined, and the exposure knee
- **One row:** LVAL, DVAL and the data bus within a row
- **Clock domains:** counting in one clock and reading in another
- **Data lines:** a running OR and AND that test all ten lines
- **Registers:** the registers to read, then set

## Takeaways

Known answers, power-up order, defaults, SPI, multi-byte values, parallel buses, clock domains, counters, hardware vs software timing, edge detection, checking a command's effect, silent failures, stuck lines, predicting before measuring, and pipelining.

The course handouts and datasheets are not included.
