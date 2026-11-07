# ECE 437 Labs

Interactive study pages for ECE 437, one tab per lab:

- **Midterm:** bring up the LSM303DLHC accelerometer on I²C bus 1 and prove its readings are real, using gravity as the referee (Part 1), then measure the capacitance of both I²C buses from their rise times (Part 2).
- **Lab 8:** drive a stepper motor from the FPGA, and use the accelerometer on the rotating arm as a protractor to count the steps it loses.

**Live page: [thattony.github.io/sensor](https://thattony.github.io/sensor/)**

It's a single `index.html` file with no build step and no dependencies. Open it in a browser, or serve the folder:

```bash
python3 -m http.server 8771
```

## Midterm

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

## Lab 8

- **Rig and home:** 300° of travel between hard stops, and why home goes at the centre
- **Tilt:** how tilting the shaft turns gravity into an absolute angle, φ = atan2(−a_t, a_r)
- **Read when stopped:** centripetal ω²r against the gravity signal at three step rates
- **Driver:** EN/DIR on the Pmod DHB1, disable before changing direction, flyback
- **Step generator:** the four full-step coil states
- **Stall protection:** chunked moves checked against the measured angle, plus a step-count limit
- **Protractor, lost steps, hysteresis, ringing and ramp, into the stop:** the measurements, with example plots

The course handouts and datasheets are not included.
