# LSM303 Bring-Up Guide

An interactive study page for ECE 437 Midterm Part 1: bring up the LSM303DLHC accelerometer on I²C bus 1 and prove its readings are real, using gravity as the referee.

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
- **Task 1a:** known-answer test
- **Task 1b:** configure, then burst read, convert and print in a loop
- **Task 1c:** the six-orientation gravity check
- **Task 1d:** the timing budget for 100 samples/s, and using STATUS_REG_A to prove each sample is new

The course handouts and the ST datasheet are not included.
