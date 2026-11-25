# Notes

## Links

| Link | How | Used for |
|---|---|---|
| PC ↔ FPGA | FrontPanel over USB: WireIn / WireOut, TriggerIn, BTPipe | commands, readings, frames |
| FPGA ↔ sensors | I²C (two buses) | five board sensors |
|  | SPI (settings) + parallel or LVDS (pixels) | CMV300 image sensor |
| FPGA ↔ actuator | PWM / DIR pins to the DHB1 | gearmotor |
| FPGA ↔ memory | DDR3 | frame buffer |
| PC ↔ bench instruments | pyvisa (SCPI over USB) | power supply, scope |

## Sensors

| Bus | Sensor | Measures |
|---|---|---|
| I²C 0 | ADT7420 | temperature |
| I²C 0 | HTS221 | humidity |
| I²C 0 | LPS35HW | pressure |
| I²C 1 | AD7156 | capacitance |
| I²C 1 | LSM303DLHC | accelerometer + magnetometer |
| SPI + video lines | CMV300 | images (parallel / LVDS) |

## What each lab did

| Lab | What it did |
|---|---|
| 2 | FrontPanel basics: send two numbers, read back the sum and difference |
| 3 | an FSM (traffic light) on the FPGA, with a testbench |
| 4 | bench instruments from Python: swept a power supply across a diode, read the scope |
| 5 | hand-written I²C FSM, first reading: ADT7420 temperature |
| 6 | finished the I²C FSM with a generator tool; found all six devices on both buses |
| 7 | noise and averaging: heated the ADT7420 with a resistor, measured its noise floor |
| Midterm | Part 1: LSM303 accelerometer bring-up. Part 2: rise time → bus capacitance of both buses |
| 8 | gearmotor (PWM/DIR) + LSM303 accelerometer as a protractor |
| 9 | camera: SPI configuration + parallel video, measured with counters |
| 10 | camera frames into DDR3, out over the USB pipe |
| Final | 350 fps camera over LVDS, characterized (noise, gain, QE); all six I²C sensors on a hardware schedule with one timebase; camera on the motor arm tracking a bright target, with latency and an error budget |

## Frame rate

**Frame rate = 1 ÷ time per frame.** Each frame goes through three stages:

exposure (light collects) → FOT, 0.195 ms (overhead) → readout (pixels sent out)

**Lab 9 numbers:**

| Mode | Time per frame | fps |
|---|---|---|
| one frame at a time: exposure + FOT + readout = 6.34 + 0.195 + 12.69 ms | 19.2 ms | 52 |
| pipelined: expose the next frame while the current one reads out, period = FOT + max(readout, exposure) | 12.9 ms | 77 |

## Failure symptoms

| Break | Symptom | Why | Find / prevent |
|---|---|---|---|
| reg 69 cleared | CLK_OUT meter reads 0, no LVAL | CLK_OUT isn’t routed to its pin, so no pixel clock reaches the FPGA | Find: WireOut 0x27 reads 0. Prevent: write reg 69 = 9 and read it back before anything else |
| Number_frames = 0 | meter reads 25,000 (clock fine) but 0 LVAL edges | the camera runs but is told to take no frames | Find: 0x27 ≈ 25,000 but LVAL edges (0x25) stay 0. Prevent: write regs 55/56 ≥ 1 and read them back |
| FRAME_REQ < 1 ms after setup | first frame missing or malformed | the camera’s PLL hasn’t settled yet | Find: first frame’s rows (0x26) ≠ 488 while later frames are fine. Prevent: wait ≥ 1 ms after the register upload before FRAME_REQ |
| packer count cleared when LINE_VALID rises | slanted image: each row shifts one more pixel | the first pixel of the row arrives in that same clock cycle, so the clear overrides it and the pixel is lost; the row comes up 1 pixel short and everything after it shifts by one | Find: the test pattern, where every pixel has a known value, so one wrong pixel is a mismatch you can count (a real scene still looks plausible). Prevent: hold the count at 0 while LINE_VALID is low |

## Data line check (OR / AND)

It’s not required. It’s a cheap check for designs with many parallel data wires, where any one wire can be open, shorted, or mapped to the wrong pin. That makes it common for camera ports, ADC buses and memory buses during board bring-up.
