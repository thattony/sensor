# LSM303 Bring-Up Guide

An interactive study page for ECE 437 Midterm Part 1 (bring up the LSM303DLHC accelerometer on I²C bus 1 and prove its readings are real, using gravity as the referee) and Part 2 (measure the capacitance of both I²C buses from their rise times).

**Live page: [thattony.github.io/sensor](https://thattony.github.io/sensor/)**

It's a single `index.html` file with no build step and no dependencies. Open it in a browser, or serve the folder:

```bash
python3 -m http.server 8771
```

## What's inside

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

The course handouts and the ST datasheet are not included.
