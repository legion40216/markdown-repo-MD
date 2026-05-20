# Document Structure

## Contents
- [Topic](#topic)
- [Questions](#questions)
- [Knowledge and Facts](#knowledge-and-facts)
- [Background](#background)
- [Concepts](#concepts)
- [Tips](#tips)
- [Notes](#notes)
- [Glossary](#glossary)
- [References](#references)
- [Article](#article)

---

# Topic

Could we reach a **1 THz processor**, and if so, **is there a limit on clock speed?**

---

## Questions

1. **Is there a limit on clock speed?**
2. **Could we reach a 1 THz processor?**

---

## Knowledge and Facts

> "Transistors were millimeters across; today they are less than 5 nm."
> — Wikipedia, CPU

> "Processing power roughly doubled as transistors shrank."
> — Wikipedia, CPU

><a id="megahertz-myth"></a>"From approximately 1995 to 2005, Intel advertised its Pentium mainstream processors primarily on the basis of clock speed alone, in comparison to competitor products from AMD. Press articles had predicted that computer processors may eventually run as fast as 10 to 20 gigahertz in the next several decades. This continued up until about 2005, when the Pentium Extreme Edition was reaching [thermal dissipation](#thermal-dissipation) limits running at speeds of nearly 4 gigahertz."
> — Wikipedia, Megahertz myth

> "The first fully mechanical digital computer, the Z1, operated at 1 Hz (cycle per second) clock frequency and the first electromechanical general purpose computer, the Z3, operated at a frequency of about 5–10 Hz."
> — Wikipedia, Clock rate

> "These records were broken in 2025 when an Intel Core i9-14900KF was overclocked to 9.12 GHz."
> — Wikipedia, Clock rate

> "The highest boost clock rate on a production processor is the i9-14900KS, clocked at 6.2 GHz, which was released in Q1 2024."
> — Wikipedia, Clock rate

> "In [CMOS](#cmos) circuits, gate capacitances are charged and discharged continually... energy is wasted in the driving transistors."
> — Wikipedia, Clock signal

>"See how FinFET (3D transistors) or Gate-All-Around (GAA) architectures try to counter [Subthreshold Leakage](#the-wall-subthreshold-leakage)

>"At 3nm you have to also bump voltage up slightly to deal with **Variability**, transistors are so small that atomic-level manufacturing imperfections cause them to behave inconsistently. A small voltage boost gives more margin for reliability. This is called **voltage guardbanding** — padding the voltage slightly above the theoretical minimum just to ensure nothing misbehaves. It costs power but buys reliability."

>"Reaching 1 THz (1,000 GHz) requires switching speeds in the picosecond range ($10^{-12}$ seconds)"

>"To build a flip-flop that operates at 1 THz, the individual transistors must have a "unity-gain cutoff frequency" ($f_T$) significantly higher than 1 THz (usually $2\times$ to $3\times$ higher) to account for the feedback loops required in a flip-flop circuit, so you would need a transistor to operate above 1 THz to actually run on 1 THz. If you tried to build a $1\text{ THz}$ flip-flop using transistors with an $f_T$ of exactly $1\text{ THz}$, the internal delays would be so large that the signal wouldn't stabilize before the next clock pulse arrived, leading to metastable states or total failure."

><a id="current-material-limits"></a>"Comparison to Current Tech
To give you a sense of the gap:
| Device Type | Typical Clock Speed |
| :--- | :--- |
| High-End CPUs | 5 – 6 GHz |
| Experimental Silicon Logic | ~100 GHz |
| Specialized InP Oscillators | 300 – 800 GHz |

><a id="spending-gains"></a>"For most of the 80s and 90s, the industry chose to spend almost all the gains on higher frequency, because clock speed was the number on the box that sold chips. ARM processors for phones were deliberately designed to prioritize power efficiency over raw clock speed. They scaled down voltage aggressively and kept frequency modest. That's why your phone doesn't burn your hand despite having billions of transistors."

><a id="air-cooling"></a>"Air cooling hits a practical ceiling around 150–200W. Beyond that you need exotic solutions — liquid cooling, phase change cooling — which are impractical for mainstream products."

><a id="power-wall"></a>"In the early 2000s, manufacturers pushed frequency ($f$) as high as possible. However, they hit a limit because the Power Density became so high that the chips would literally melt or require industrial-grade cooling. This is why the industry shifted away from raw clock speed and toward:
1. [Multi-core processing](#multi-core-processing)
2. [Instruction per Cycle (IPC)](#instruction-per-cycle)"

><a id="5"></a>"The power dissipated by a circuit increases with the square of the voltage applied, so even small voltage increases significantly affect power."
> — Wikipedia, Dynamic voltage scaling"

---

## Background

### Transistor scaling

><a id="transistor-scaling-limit"></a> "The physical limits to transistor scaling have been reached due to source-to-drain leakage, limited gate metals and limited options for channel material. Other approaches are being investigated, which do not rely on physical scaling. These include the spin state of electron spintronics, tunnel junctions, and advanced confinement of channel materials via nano-wire geometry."
> — Wikipedia, Moore's Law

> "The complexity of an integrated circuit is bounded by physical limitations on the number of transistors that can be put onto one chip, the number of package terminations that can connect the processor to other parts of the system, the number of interconnections it is possible to make on the chip, and the heat that the chip can dissipate."
> — Wikipedia, Microprocessor

### Heat and power

> <a id="short-circuit-power"></a>"In [CMOS](#cmos) circuits, gate capacitances are charged and discharged continually... energy is wasted in the driving transistors."
> — Wikipedia, Clock signal

> <a id="dynamic-power"></a>"The switching power dissipated by a chip using static [CMOS](#cmos) gates is α · C · V² · f, where C is the capacitance being switched per clock cycle, V is the supply voltage, f is the switching frequency, and α is the activity factor."
> — Wikipedia, Dynamic voltage scaling

> "Higher supply voltages result in faster slew rate, which allows for quicker transitioning through the MOSFET's threshold voltage. Quicker transitioning afforded by higher supply voltages allows for operating at higher frequencies."
> — Wikipedia, Dynamic voltage scaling

> "**Peak performance** of any system is essentially limited by the amount of power it can draw and the amount of heat it can dissipate."
> — Wikipedia, Performance per watt

> "The total amount of [leakage currents](#leakage-current) tends to inflate for increasing temperature and decreasing transistor sizes."
> — Wikipedia, Processor power dissipation

### Propagation delay and chip size

> "An electric signal travelling through a wire has a [propagation delay](#propagation-delay) of ca. 1 nanosecond per 15 centimetres."
> — Wikipedia, Propagation delay

> "This delay is the major obstacle in the development of high-speed computers and is called the interconnect bottleneck in IC systems."
> — Wikipedia, Propagation delay

> "As the clock rate of a circuit increases, timing becomes more critical and less variation can be tolerated if the circuit is to function properly."
> — Wikipedia, [Clock skew](#clock-skew)

### The multicore shift

><a id="multicore-shift"></a>"While manufacturing technology improves, reducing the size of individual gates, physical limits of semiconductor-based microelectronics have become a major design concern. These physical limitations can cause significant heat dissipation and data synchronization problems. Various other methods are used to improve CPU performance."
> — Wikipedia, Multi-core processor

---

## Concepts

### What a clock cycle actually is
Every clock cycle, a signal travels through the entire chip and forces every storage element (flip-flops, latches) to switch state simultaneously. Switching means charging and discharging capacitors — which wastes energy as heat. More cycles per second = more switching events per second = more heat per second. This is not a cooling engineering problem. It is physics. The heat is a direct byproduct of the computation itself.

### Three walls, not one
There are three independent power sources in every CPU:
- **[Dynamic power](#dynamic-power)** — capacitors charging/discharging every clock cycle (CV²f)
- **[Short-circuit power](#short-circuit-power)** — transistors briefly conduct simultaneously during each transition
- **[Leakage current](#leakage-current)** — current that bleeds through transistors even when idle; gets worse as transistors shrink and temperature rises.

At smaller transistor sizes and higher temperatures, all three worsen simultaneously. Pushing toward 1 THz triggers all three at once.

### The power wall and the multicore shift
Around 2004–2005, the industry hit what's called the power wall — the point where pushing clock speed higher consumed so much power that the heat became unmanageable. Performance per watt became the metric that mattered more than raw clock speed. Instead of one fast core, manufacturers started putting multiple efficient cores on one chip. More work done per watt, less heat per unit of performance. This is why clock speeds plateaued near 4 GHz and haven't moved much since.
*→ see [power wall](#power-wall), [multicore shift](#multicore-shift)*

### The Size Trap — Where Propagation Delay and Transistor Limits Collide

Signal [propagation delay](#propagation-delay) has always existed — it is not a new barrier at 1 THz. But it becomes the deciding constraint when combined with transistor limits.

At 1 THz, one clock cycle lasts 1 picosecond. Electrical signals travel at roughly 0.6× the speed of light through wire — about 1 nanosecond per 15cm, or 1 picosecond per 0.15mm. This means the entire chip must be smaller than ~0.15mm for a signal to cross it within one cycle.

To fit a useful processor into 0.15mm, transistors would need to be smaller than anything physically manufacturable. At sizes below ~2nm, [quantum tunneling](#quantum-tunneling) causes electrons to pass through transistor walls regardless of switching state — the transistor ceases to function as a reliable switch. *→ see [Quantum Tunneling](#quantum-tunneling)*

---

### Three Material Limits That Appear Even If You Solve the Size Problem

Solving the geometry gets you to the door. These are the locks on it.

**1. Electron Saturation Velocity**

Electrons don't move instantly through a material — they have a maximum speed called the saturation velocity, a hard ceiling set by the crystal lattice itself. No amount of additional voltage pushes them faster beyond this point. To switch a transistor at 1 THz, the gate length must be short enough that electrons can cross it within a fraction of a picosecond. At the scales required, we are not just engineering around this limit — we are running directly into it.

A 1 THz flip-flop isn't a faster version of what's in your PC. It requires a different class of physics. We can generate 1 THz signals using oscillators, but performing stable digital logic at that frequency — where a transistor must reliably hold a 0 or a 1 — is a separate and much harder problem. *→ see [current material limits](#current-material-limits)*

**2. RC Delay — The Wire Problem**

Even if a transistor flips in 0.5 picoseconds, the signal still has to travel down a wire to reach the next gate. As transistors shrink, the wires connecting them also get thinner — and thinner wires have higher resistance. The result is RC delay: resistance and capacitance in the wire slow the signal down independent of how fast the transistor itself switches. A signal might take 5 picoseconds to travel down a microscopic copper wire even after the transistor has already done its job. At 1 THz, a wire that is just 1 micrometer long starts behaving like an antenna — leaking energy and generating enough electrical noise that the receiving gate can no longer reliably distinguish a 1 from a 0.

**3. Heat Density**

When a transistor switches, moving electrons generates heat. At 1 THz with transistors packed at modern densities, the power dissipated per unit area would be comparable to the surface of the sun. A single flip-flop can be made to run at 200 GHz in a lab because it can be cooled in isolation. Pack billions of them together switching at that rate and the silicon substrate melts. This isn't a cooling engineering problem waiting for a better heatsink — it is a fundamental consequence of how much energy switching requires at that frequency. *→ see [Dynamic Power](#dynamic-power)*

---

### Quantum Tunneling

At transistor sizes below ~2nm, the insulating walls of the transistor are only a few atoms thick. At that scale, quantum mechanics allows electrons to "tunnel" through the barrier — passing through a wall they classically should not be able to cross, regardless of the switching state. A transistor that cannot reliably stay OFF loses its function as a binary switch entirely. The flip-flop built from it can no longer hold a stable 0 or 1.

This is not a manufacturing defect that better fabrication can fix. It is a consequence of the wave-like nature of electrons at atomic scales. Shrinking further doesn't improve the situation — it makes tunneling more probable. *→ see [Leakage Current](#leakage-current)*

---

### Clock Speed Performance

Clock speed performance isn't controlled by a single variable — it emerges from the interaction of four tightly coupled parameters: **Frequency, Voltage, Power Density, and Heat.** Pull on one and the others move.

#### The Four Parameters

**Frequency** is the most direct measure of speed — how many cycles a processor executes per second (GHz). Higher frequency means faster data processing, but it doesn't come free.

**Voltage** is the electrical pressure required to switch transistors on and off. To achieve higher frequencies, voltage must increase so that electrical signals can clear the gates fast enough without errors. This leads directly to the danger: voltage has a *squared* relationship with power consumption ($P \propto V^2$). Even a small bump in voltage causes a disproportionate spike in heat.

**Power Density** is the amount of power dissipated per unit of area on the silicon die. As chips shrink (7nm, 5nm, 3nm), more transistors are packed into a smaller space. A chip consuming 100W across 100mm² is far harder to cool than the same 100W spread across 400mm².

**Heat** is the waste product of the interaction between voltage and frequency. It is not a cooling engineering problem — it is physics. Every clock cycle forces every storage element to switch state, which means charging and discharging capacitors, which wastes energy as heat. More cycles per second means more switching events per second means more heat per second.

The chain runs in one direction: push Frequency → must raise Voltage → Power Density spikes → Heat becomes unmanageable → system throttles everything back down.

---

#### The Formula: $P = C \cdot V^2 \cdot f$

This is the mathematical heart of how processors are designed and throttled.

- **$P$ (Power):** Total dynamic power dissipated as heat. As $P$ increases, heat and power density become the primary walls blocking further performance.
- **$C$ (Capacitance):** The physical characteristics of the chip — its wires and transistors. Shrinking transistors lowers $C$, but as they get closer together, leakage becomes a new problem. *→ see [Leakage Current](#leakage-current)*
- **$V^2$ (Voltage Squared):** The most dangerous term. Doubling voltage doesn't double power — it quadruples it. This is why the heat problem gets catastrophically worse as frequency climbs. It's not a gradual relationship; it's a square relationship. **If you could halve V, power would drop by 4×. If V is stuck, you lose that 4× saving every single generation while f keeps climbing** — a compounding disaster across multiple process nodes.
- **$f$ (Frequency):** Power scales linearly with frequency. Double the clock speed, double the power. But to get that higher $f$, you almost always have to raise $V$ as well, which triggers the squared penalty on top of the linear one. *→ see [Dynamic Power](#dynamic-power)*

---

#### The Two Directions of Voltage

This is where the concept becomes a trap, and it has two sides.

**Increasing frequency requires raising voltage.** When you push a chip to run faster, clock ticks get shorter. Transistors now have less time to fully switch from 0 to 1 before the next tick arrives. To compensate, you increase voltage — shoving electrons harder so they move fast enough to finish switching in time. More voltage means more speed, but the squared penalty means power explodes.

**The only escape is to lower voltage first.** This is the entire logic of Dennard Scaling: shrink the transistor → lower the threshold voltage → lower the operating voltage → free up headroom in the power budget → spend that headroom on higher frequency. The chip ends up faster while consuming roughly the same power as the previous generation. Voltage reduction wasn't a side effect of scaling. It was the *prerequisite* for every frequency gain.

Which makes what happens next a genuine catastrophe.

---

#### <a id="subthreshold-leakage"></a>The Wall: Subthreshold Leakage

Those thermally agitated electrons aren't just sitting there quietly. They're randomly bouncing around with kinetic energy. At room temperature, that energy is about **26 meV** — a small but non-zero kick that every electron has for free, just from existing at room temperature.

If threshold voltage is set high enough, that 26 meV kick is insignificant — like a small wave against a tall dam. The transistor stays reliably off.

But if you keep scaling the threshold voltage down — *as Dennard requires, as you must, in order to lower operating voltage, in order to increase frequency* — the dam gets shorter. At some point the random thermal kicks become comparable to the threshold energy, and electrons start accidentally surging over it, switching the transistor on when nothing told it to. This is **Subthreshold Leakage**: the chip burns power and generates heat even when it's doing absolutely nothing.

*An analogy:* Imagine a ball in a valley between two hills. The hill height is your threshold voltage. Thermal noise is constant random vibration of the ground.

- **High hill** → ball stays in the valley no matter how much the ground shakes. Reliable.
- **Low hill** → the random shaking eventually bounces the ball over. Unpredictable switching.

Dennard Scaling keeps telling you to make the hill shorter — because you *need* a shorter hill to lower operating voltage, because you *need* lower operating voltage to gain frequency without melting the chip. That chain of dependencies is the trap. Thermal noise is the shaking you can't turn off without cooling the chip to near absolute zero.

So the whole strategy collapses at once: you can't lower the hill → can't lower the voltage → no power budget left to spend on frequency. This is why CPUs today sit between 3–5 GHz rather than the 10–20 GHz that 1990s projections promised.

---

#### What to Do With a Transistor Shrink

When a transistor shrinks, the gains don't spend themselves automatically. Engineers face a decision:

- **Higher frequency** — switch faster, extract more raw performance
- **Lower voltage** — switch at the same speed but consume less power
- **Both partially** — split the gains between speed and efficiency

This tradeoff is why **Performance per Watt** has become the defining metric of modern chip design — not raw clock speed. *→ see [Spending gains](#spending-gains)*

---

### Moore's Law and Dennard Scaling

For decades, two laws ran in parallel and the entire semiconductor industry depended on both of them holding simultaneously.

**Moore's Law** observed that the number of transistors on a chip doubles roughly every two years as manufacturing improves. More transistors means more computational capability in the same area.

**Dennard Scaling** said that as each transistor shrinks, it also needs proportionally less power — so even though you're adding more transistors, the total power density across the chip stays flat. Voltage, current, and switching energy all scale down together with the transistor dimensions.

Moore gives you more transistors. Dennard ensured all those extra transistors didn't burn the chip.

They were not independent. Dennard was the reason Moore's Law was *usable*. Without Dennard, doubling the transistor count would have doubled the heat. With both working together, you got more performance for free — faster, denser, and no hotter than the previous generation.

---

#### When One Broke and the Other Didn't

Around 2004, Dennard Scaling stopped working. Moore's Law continued.

The reason Dennard broke is the thermal noise floor described in [Subthreshold Leakage](#subthreshold-leakage). Voltage has a hard minimum — push it below the point where the threshold voltage becomes comparable to 26 meV of thermal noise, and transistors start switching randomly. So voltage stopped scaling down with each generation. It basically flatlined. Modern chips at 3nm run at roughly the same voltage as chips from 10 years ago at 22nm.

But Moore's Law kept going. Transistors kept shrinking. Which created a specific and serious problem.

With voltage stuck, look at what happens to the formula $P = C \cdot V^2 \cdot f$:

- **$C$** kept falling — smaller transistors need less charge to switch ✅
- **$V$** stopped falling — thermal noise floor hit ❌
- **$f$** kept climbing — the industry kept pushing for more speed ❌

$C$ shrinking is a linear saving. $V$ being stuck is a squared loss. The math is unforgiving: if $V$ could have halved, power would have dropped by 4×. $C$ halving saves only 2×. The $C$ savings don't come close to compensating for $V$ being frozen — and meanwhile frequency kept climbing, adding to the load on top.

The result: power density started rising every generation instead of staying flat. Each new chip was hotter than the last.

---

#### The Pentium 4 Prescott — Where the Wall Became Visible

The clearest example of this collapse is Intel's Pentium 4 "Prescott" in 2004:

- 90nm process, pushing toward 4 GHz
- Power consumption reaching ~115W
- Next generation projecting 150W+
- The trajectory beyond that was pointing toward 200W+

The fundamental constraint was cooling. You can only extract so much heat from a small piece of silicon regardless of how good the heatsink is. Intel was approaching that ceiling fast. The architecture was eventually cancelled, and the industry pivoted almost overnight.

This wasn't a gradual transition. The thermal wall hit suddenly enough that it ended the clock speed era within a single product generation.

---

#### The Irony

The thing the industry was chasing — higher frequency — is precisely what made the thermal wall inevitable. Frequency kept $V$ stuck at a high level because high-speed switching requires more electrical headroom. If the industry had held frequency flat and let voltage continue scaling down, power density would have stayed flat or dropped. The transistors were physically capable of running efficiently at lower frequencies.

But nobody was selling chips on "same speed, less power." They were selling GHz. So they kept pushing $f$ up, and the equation punished them for it the moment $V$ had nothing left to give.

---

### What Shrinking Buys Today

Transistors are still shrinking. Manufacturing has continued from 22nm down to 7nm, 5nm, 3nm, and beyond. But what shrinking *delivers* has fundamentally changed since Dennard broke.

When both laws were working, shrinking a transistor meant three things at once: more transistors, lower power per transistor, and higher frequency at the same power budget. You got all three for free every generation.

Today, shrinking delivers almost entirely one thing: **density.** More transistors in the same area.

Since voltage is frozen, the "lower power per transistor" relationship that Dennard promised is largely gone. Each individual transistor isn't meaningfully cheaper to run than it was several nodes ago. What you're getting is more of them in the same space.

---

#### What Density Actually Buys

More transistors in the same area means engineers can now choose how to spend that budget:

- **More cores** — instead of one faster processor, put several parallel ones on the same die
- **Bigger caches** — more on-chip memory means less waiting on slower RAM
- **Specialized hardware** — AI accelerators, media engines, signal processors, security enclaves — fixed-function silicon that does one job extremely efficiently rather than general-purpose logic doing it slowly

This is the multicore era and the era of heterogeneous chips. Apple's M-series, AMD's Zen architecture, Intel's hybrid designs — all of them are expressions of the same underlying reality: raw clock speed stopped being the lever, so the industry found different levers.

---

#### The Honest Tradeoff

This is a real improvement in what chips can do. But it's worth being clear about what was lost.

In the Dennard era, performance scaled *automatically*. Write any program, run it on next year's chip, it runs faster — for free, no changes needed. Single-threaded performance climbed every generation.

Today, extracting the gains from more cores requires software that is written to use them. A program that runs on one thread doesn't get faster just because the chip has 16 cores. Specialized accelerators only help workloads that match what they were built for. The performance gains are real, but they are no longer automatic or universal.

The shift from "faster clock" to "more parallel hardware" was a fundamental change in how performance is delivered — and in what programmers have to do to capture it.

---

### DVFS — Dynamic Voltage and Frequency Scaling

The voltage floor isn't a fixed wall. It moves with frequency.

This is the insight behind **DVFS — Dynamic Voltage and Frequency Scaling** — the technique that every modern chip uses to manage power at runtime.

---

#### Why the Floor Moves

The reason voltage has a minimum is that at high frequencies, transistors need to switch fast. The clock tick is short, so the transistor needs enough electrical pressure to reliably cross the threshold before the next tick arrives. More speed demands more headroom.

But if you slow the switching down — lower the frequency — the transistor has more time to cross the threshold. It can do so with less voltage. The minimum stable voltage drops with frequency.

The thermal noise floor at 26 meV is the absolute basement — you cannot go below that regardless of frequency. But between that floor and the operating voltage of a chip running at full speed, there is significant room. DVFS exploits that room in real time.

---

#### How It Works in Practice

Every modern chip — phone, laptop, desktop — continuously monitors its current workload and adjusts both voltage and frequency together:

- **Light workload** (reading email, browsing) → drop frequency → drop voltage → power falls dramatically
- **Heavy workload** (compiling code, running a game) → raise frequency → raise voltage → full performance on demand

Because power scales with $V^2$, dropping both $f$ and $V$ together saves power far more than dropping frequency alone would. The squared term means even a modest reduction in voltage has an outsized effect on the total power budget.

This is exactly what happens when your phone gets warm under load and cools down when idle — the chip isn't running at a fixed speed. It's constantly negotiating between performance and power at a timescale of milliseconds.

---

#### What This Means for the "Voltage is Stuck" Problem

DVFS reframes the Dennard collapse slightly. The problem wasn't that voltage *couldn't* go lower — it's that the industry kept pushing frequency up, which kept the *minimum stable voltage* high, which kept power high.

Voltage and frequency are coupled. Lower one and you can lower the other. The industry's mistake was treating frequency as the only dial worth turning, which left voltage with nowhere to go. DVFS is the acknowledgment that both dials matter and that the right answer changes depending on what the chip actually needs to do at any given moment.

---

## Tips

Answering the topic all you need is **"how far have we gotten?"** and **"why can't we go further?"** That's it. Everything else just fills those two questions out.

---

## Notes

Clock speed contributes to **processing power**, but processing power does **not depend solely on clock speed**.

Other factors influencing processing power include:

- Number of cores
- Instruction efficiency
- Cache design
- CPU architecture

> "A semiconductor device's processing speed depends on a variety of factors, including, but not limited to, its clock speed, microarchitecture, the kind of software it's running, and the bandwidth, latency and size for each level of its memory."
> — Wikipedia, Overclocking

---

## Glossary

<a id="multi-core-processing"></a>
**Multi-core processing** — Running four cores at a lower $f$ and $V$ is much more power-efficient than running one core at a massive $f$.

<a id="instruction-per-cycle"></a>
**Instruction per Cycle** — Improving how much work is done in a single "tick" of the clock, rather than just making the clock tick faster.

<a id="thermal-throttling"></a>
**Thermal Throttling** — A protective mechanism where the chip automatically reduces its clock speed and voltage when it detects its temperature approaching a dangerous limit. Trades performance for survival — the chip runs slower but doesn't burn out.

<a id="throttle"></a>
**Throttled/Throttle** — When a chip is forced to reduce its clock speed due to thermal or power constraints. A throttled chip is running below its rated speed because the system can't sustain the heat or power draw at full speed.

<a id="mosfet"></a>
**MOSFET (Metal-Oxide-Semiconductor Field-Effect Transistor)** — The fundamental switching device in all modern processors. A MOSFET acts as a gate-controlled switch: when voltage applied to the gate exceeds the threshold voltage, a conducting channel forms and current flows; below the threshold, the channel closes and current stops. Virtually every transistor in a modern CPU is a MOSFET.

<a id="thermal-noise"></a>
**Thermal Noise** —
The hotter something is, the more violently they shake. Even at room temperature (~300 Kelvin), electrons in a material have a constant, random kinetic energy — they're never perfectly still. This random agitation is thermal noise.

<a id="moore-law"></a>
**Moore's Law** —
is a manufacturing observation — it's about how precisely we can etch silicon. It kept going because lithography, materials science, and fabrication tools kept improving. Transistors kept shrinking geometrically.

<a id="dennard-scaling"></a>
**Dennard Scaling** —
is a physics relationship — it only works if you can proportionally shrink voltage alongside the transistor.

<a id="thermal-dissipation"></a>
**Thermal dissipation** — the rate at which a component sheds heat into its surroundings. If a chip generates heat faster than it can dissipate it, it throttles its clock speed or shuts down to avoid permanent damage.

<a id="cmos"></a>
**CMOS (Complementary Metal-Oxide-Semiconductor)** — the transistor technology used in virtually all modern processors. CMOS gates dissipate power by charging and discharging capacitors on every clock cycle.

<a id="leakage-current"></a>
**Leakage current** — current that bleeds through a transistor even when it is switched off. Gets significantly worse as transistors shrink below ~7nm and as temperature rises.

<a id="propagation-delay"></a>
**Propagation delay** — the time it takes for an electrical signal to travel from one point to another through a wire or circuit. In copper wire, roughly 1 nanosecond per 15cm.

<a id="clock-skew"></a>
**Clock skew** — the phenomenon where the same clock signal arrives at different parts of a chip at slightly different times. At higher frequencies the tolerance window shrinks until skew causes data errors.

<a id="quantum-tunneling"></a>
**Quantum tunneling** — a quantum mechanical effect where an electron passes through a barrier it classically shouldn't be able to cross. At transistor sizes below ~2nm, electrons tunnel through the transistor gate, making it impossible to switch reliably off.

<a id="power-wall"></a>
**Power wall** — the practical ceiling hit around 2004–2005 where increasing clock speed further produced more heat than cooling systems could manage. Led the industry to abandon the MHz/GHz race in favor of multicore designs.

---

## References

| # | Page | Used for |
|---|------|----------|
| 1 | Wikipedia — CPU | Transistor size history |
| 2 | Wikipedia — Clock rate | Speed records, IPC improvements |
| 3 | Wikipedia — Moore's Law | Transistor scaling limits |
| 4 | Wikipedia — Microprocessor | IC complexity limits |
| 5 | Wikipedia — Megahertz myth | Intel clock speed era, power wall |
| 6 | Wikipedia — Clock signal | Heat mechanism, capacitor switching |
| 7 | Wikipedia — Dynamic voltage scaling | V² power formula, voltage-frequency relationship |
| 8 | Wikipedia — Performance per watt | Hard limit on power and heat |
| 9 | Wikipedia — Multi-core processor | Industry response to heat limits |
| 10 | Wikipedia — Processor power dissipation | Three power sources, leakage current |
| 11 | Wikipedia — Overclocking | Clock speed vs overall performance |
| 12 | Wikipedia — Propagation delay | Signal speed in wire, interconnect bottleneck |
| 13 | Wikipedia — Clock skew | Timing tolerance collapses at high frequency |

---

## Article