# Questions

## Which parts of the labs are only for finishing the lab, and which last?

| Kind | Examples |
|---|---|
| Only for the lab (look it up when needed) | register values, endpoint maps, sensor timing constants, rig details, report plot formatting |
| Lasting | bandwidth and memory budgets before writing RTL, backpressure and dropped frames, protocols read at the waveform level, and everything in [Takeaways](takeaways.md) |

## Why do the labs lean so hard on noise and measurement, and is it overdone?

It’s a sensors and instrumentation course, so measurement is the subject. The habits last; the exact numbers mostly don’t.

| Keep | Overdone for most jobs |
|---|---|
| know the noise floor before trusting a number | repeated sweeps for error bars on a lab rig |
| digital lines are analog: rise time ≈ RC sets the top bus speed | bus capacitance to the pF from a scope trace |
| random vs systematic error: averaging N samples cuts random noise by √N, not an offset | polished report plots |

| Job | How much it matters |
|---|---|
| RTL / DV / ASIC | noise math rarely; measurement becomes coverage and assertions |
| Board / embedded / FPGA | a lot: noise, ADC resolution, rise time, bring-up order; regulated products (e.g. medical) need measured margins against the spec |

## What does “works on the bench, fails in the enclosure” mean?

The enclosure is the product’s case. The same board behaves differently once it’s installed.

|   | Bench | Enclosure |
|---|---|---|
| Wiring | short wires | cable harness, more capacitance |
| Bus | 1–2 devices | more devices |
| Noise nearby | little | motor, power supply, radio |
| Temperature | room | warmer, heat trapped |

> **Example.** I²C at 400 kHz, 2.2 kΩ pull-ups. Bench: 50 pF, rise ≈ 0.85 · RC ≈ 93 ns. A 30 cm cable adds 150 pF: 200 pF, rise ≈ 373 ns, over the 300 ns limit. Result: occasional NACKs or flipped bits. Fix: stronger pull-up, slower clock or a bus buffer.

## What does “turning sensor noise into resolution in the quantity you care about” mean?

A sensor measures one thing, but you want another. Its noise is in the units it measures, so convert it with the sensitivity (how much the reading changes per unit of what you want).

resolution = sensor noise ÷ sensitivity

| Measures | You want | Sensitivity | Resolution |
|---|---|---|---|
| acceleration (Lab 8) | arm angle | g · sin 15° ≈ 0.259 g per rad | 1 mg → 0.22° |
| ADC voltage (Example) | temperature | 10 mV per °C | 2.4 mV (3 steps of 0.8 mV) → 0.24 °C |
| air pressure (Example) | altitude | ≈ 12 Pa per m | 1 Pa → ≈ 8 cm |

It also guides design: at a 10° tilt the signal shrinks, and the same 1 mg becomes 0.33°.

## Do I need to measure bus capacitance myself, or just trust the datasheet?

Both: design from the datasheet, measure to check.

- Design: add up pin capacitances plus a trace and cable estimate, pick the pull-up.
- Verify on the first board: scope the rise time and check it’s under the limit with margin. Usually that’s enough.
- Back-calculate C only when debugging, e.g. a bus that fails in the enclosure.

Datasheets can be wrong too. In Lab 9, an older CMV300 datasheet gave reg 69 = 8 (it’s 9), and reg 83’s default PLL band was wrong for 25 MHz, putting every timing number off by 47% with no other symptom.
