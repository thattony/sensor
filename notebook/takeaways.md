# Takeaways

Habits and patterns from these labs that apply to most embedded and FPGA work, whatever the chip.

## Test a known answer

Before trusting any measurement from a new chip, read something whose value you already know. If that comes back right, the wiring, the bus, the timing and the bit order all work. If it doesn't, nothing after it can be trusted.

- **ID register**: many chips have one that always reads a fixed value listed in the datasheet.
- **Reset defaults**: registers start at documented values after reset.
- **Write, then read back**: write a value, read it again, compare.

> **Example.** Write 0x5A and read back 0xB4: every bit moved one place left, so the read samples one clock late.

## Power up in order

A complex chip needs its supplies, then a running clock, then reset released, then its settings. Reset clears the registers, so settings written before it are lost, and a chip with no clock can't come out of reset cleanly. A PLL also needs time to lock before its clock can be used.

- Supplies stable → clock on → hold reset → release reset → wait → configure → wait for lock or ready → start.
- The datasheet gives this as a timing diagram. Every wait in it is a minimum.
- Getting it wrong rarely gives an error. The chip just doesn't answer, or answers with nonsense.

## Don't trust defaults

A default suits a typical case, not necessarily your board, and datasheets have typos that a later revision corrects. So read the revision history and the errata, and measure anything you only assumed: a clock frequency, a mode, a gain.

> **Example.** A PLL set for the wrong input range may still lock, just at the wrong frequency. Everything runs and every time measurement is off by the same ratio. Only measuring the clock shows it.

## How SPI works

SPI has four wires: a clock and a chip select from the master, one data line out of the master and one back. Each transfer is a fixed number of bits, usually a read/write bit, an address and a data byte, sent MSB first.

On each data line one side changes the bit on one clock edge and the other side samples it on the opposite edge, so the bit has half a period to settle. To set up a master, read the slave's datasheet rather than guessing a mode number:

- Find the edge the slave samples its input on. Change your output on the other edge.
- Find the edge the slave changes its output on. Sample its output on the other edge.

The same pattern of a frame plus edges appears in flash memory, ADCs, DACs and most sensors.

## Multi-byte values

A value wider than 8 bits is stored in two or more 8-bit registers. The datasheet says which byte comes first: low byte first (little-endian) or high byte first (big-endian).

> **Example.** 300 = 0x12C, stored as a low byte 0x2C and a high byte 0x01. Value = low + 256 × high. With the bytes swapped you'd get 0x2C01 = 11,265.

A one-bit shift in the receiver doubles or halves every value, which can look plausible. A known value is the only reliable check.

## Parallel buses

Fast data often comes out on several data lines at once, with its own clock and one or more "valid" lines that say when the data means something. Cameras, displays and fast ADCs use this shape.

- Sample on the clock edge opposite the one where the data changes.
- Count or store only while the valid line is high. Gaps can sit inside a line, not just between lines.
- A long low stretch on a valid line usually marks the end of a block, such as a frame.

## Crossing clock domains

A flip-flop samples its input on a clock edge. If the input changes right at that edge, the flip-flop can hang between 0 and 1 for a while (metastability). That's unavoidable when a signal comes from logic on a different, unrelated clock.

- **One bit**: pass it through two flip-flops in a row. The first may go metastable; the second gives it a full cycle to settle.
- **Many bits**: two flip-flops per bit isn't enough. Each bit may be caught on a different cycle, so the value can read half old and half new.
- **The fix**: hold the value still, flip one toggle bit, and send only that bit across. When the other side sees the toggle change, the value is stable, so it copies it.

Clock-crossing bugs are rare and random, so they're hard to find in testing. This is one of the most common FPGA interview topics.

## Measure with counters

A counter inside an FPGA can replace a scope. A counter counts events: rows, pixels, pulses. A meter counts clock ticks over a known time, which gives a frequency or a duration.

> **Example.** Open a 1 ms window timed by a known 100 MHz clock (100,000 ticks) and count edges of an unknown clock during it. 25,000 edges means 25 MHz.

The same idea is behind built-in self-test in production hardware, which checks itself with no lab equipment.

## Hardware vs software timing

A sleep in software on a PC or an OS is coarse and varies run to run, often by a millisecond or more. A counter in hardware on a 100 MHz clock is exact to 10 ns.

- Microsecond waits, or anything that must be exact: a hardware counter.
- Millisecond waits with plenty of margin: software is fine.

Splitting work this way between firmware and logic is a decision you make on almost every board.

## Edge detection

Many inputs are meant as one-time commands but may arrive as a level that stays high. Pass it on as is and the receiver may act on it again and again. An edge detector gives exactly one clock-wide pulse per rising edge: keep last cycle's value and fire when the input is high now and was low before.

```verilog
reg prev;
always @(posedge clk) prev <= in;
assign pulse = in & ~prev;   // high for one clock on each rising edge
```

If the input comes from another clock domain, bring it across with two flip-flops first.

## Check the effect

Some commands get no reply: a trigger pin, a start-conversion strobe, a pulse to a motor driver. Since there's nothing to read back, check the effect instead: data that arrives, a flag that rises, a delay you can time.

> **Example.** After a "start" pulse, time how long until the first data appears and count how much arrives. Both should match what the datasheet predicts.

## Catch silent failures

Hardware usually fails quietly. Missing clocks, wrong settings and early commands each just produce nothing, or something slightly off. List every condition that must hold, and next to each one, the measurement that would show it doesn't.

- Clock missing → a frequency counter reads 0.
- A setting not written → a register read-back shows the default.
- A command sent too early → the first result is missing or the wrong size.

That list is a bring-up checklist and a test plan.

## Find stuck lines

Keep a running OR and a running AND of every word on a bus. Start OR at all 0s and AND at all 1s.

- A bit that never turns on in OR: that line never went high, so it's stuck low or open.
- A bit that stays on in AND: that line never went low, so it's stuck high.

With varied data, OR should end all 1s and AND all 0s. This takes two registers per bus, and works on any bus: ADC data, address lines, memory.

## Predict, then measure

Work out the expected number, with a tolerance, before taking the measurement. If the two disagree, the gap tells you something: a wrong assumption, a wrong setting or a bug. A measurement with no prediction has nothing to disagree with.

## Pipelining

If a job has stages that run one after another, one item takes the sum of all stages. If the next item can start an early stage while the current one is still in a later stage, the stages overlap, and a new item finishes once per slowest stage.

> **Example.** Washing takes 30 min and drying 40 min. One load takes 70 min. With many loads, one finishes every 40 min.

Overlap hides the faster stages but doesn't make the slowest one faster. Speeding up anything except the slowest stage changes nothing. The same holds for CPU pipelines, double-buffered DMA and streaming data.
