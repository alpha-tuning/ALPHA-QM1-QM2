## Why I Designed a MOSFET-Based QM1 / QM2 Replacement

The original **STA460C** used in Honda OBD1 ECUs is an **NPN Darlington injector driver with a built-in avalanche diode**. It was designed to act as a low-side injector switch while also surviving the inductive kick generated when the injector is turned off. On paper, it is rated around **60 V**, **6 A continuous collector current**, **10 A pulse current**, and **200 mJ single-pulse avalanche energy**. :contentReference[oaicite:0]{index=0}

My goal with this project was not just to replace the package or make something that “sort of works.” The goal was to build a modern replacement that behaves like the original circuit where it matters, while using parts that are easier to source, more robust, and easier for people to understand and reproduce.

Instead of copying the original Darlington transistor approach, I chose to recreate the **function** of the STA460C using a modern **N-channel MOSFET** and a small group of support components selected to shape the behavior of the circuit.

## What I used and why

At the heart of the design is the **BSC025N08LS5**. This is an **80 V logic-level N-channel MOSFET** with very low **RDS(on)**, strong pulse capability, and solid avalanche performance. Infineon specifies **80 V drain-source rating**, **370 mJ single-pulse avalanche energy**, and notes that the device is **100% avalanche tested**. :contentReference[oaicite:1]{index=1}

That already gives this design more voltage and avalanche headroom than the original STA460C. The original part is a **60 V bipolar injector driver** with **200 mJ avalanche energy**, while the MOSFET I selected is an **80 V part** with a higher single-pulse avalanche rating. 

But the transistor alone is not what makes this work.

I added a small support network around the MOSFET so the whole circuit behaves like a proper injector driver rather than just a generic switch.

Each channel uses:

- a **47 Ω gate resistor**
- a **47 kΩ gate pulldown**
- a **15 V zener from gate to source**
- a **43 V TVS across the switch node**

Each one has a job.

The **47 kΩ pulldown** makes sure the MOSFET stays off when it is supposed to. That keeps the gate from floating and makes startup and idle behavior predictable.

The **47 Ω gate resistor** helps control the edge rate and damp ringing. MOSFETs can switch extremely fast, and while that is great, I did not want the gate being driven so hard that the circuit became noisy or unstable. The gate resistor helps the transition stay controlled.

The **15 V zener** protects the MOSFET gate oxide from transients. Gate protection matters in an automotive environment and especially in circuits that are sitting next to inductive loads.

The **43 V TVS** is one of the most important parts of the design. This is what helps the MOSFET circuit behave like the original injector driver during turn-off. When an injector is switched off, the magnetic field in the injector coil has to collapse, and that means the voltage at the switching node rises sharply. The original STA460C handled that with its internal avalanche structure. My design handles it with an external controlled clamp path.

That is the key idea behind this project:

I did not copy the original transistor topology.  
I recreated the original **behavior** using modern parts and a controlled clamp network.

## Why a MOSFET can replace a Darlington here

A lot of people see the original part is a Darlington transistor and assume the replacement also has to be a Darlington. I do not look at it that way.

What matters most in this application is not whether the silicon inside is bipolar or MOSFET. What matters is whether the circuit performs the same job:

- switch injector current reliably
- survive inductive turn-off
- clamp the flyback event in a controlled way
- provide repeatable behavior under real load

That is exactly what this design does.

The STA460C is a bipolar device, so when it is on, it operates with a **collector-emitter saturation voltage**. The datasheet shows about **0.09 V typical / 0.15 V max** at **1.5 A** with the stated base-drive condition. :contentReference[oaicite:3]{index=3}

The MOSFET works differently. Instead of a saturation voltage like a Darlington, it behaves more like a very small resistor when fully enhanced. The BSC025N08LS5 has an **RDS(on)** around **2.5 mΩ max at 10 V gate drive** and **3.3 mΩ max at 4.5 V**. :contentReference[oaicite:4]{index=4}

That means the MOSFET replacement can switch injector current with extremely low on-state loss. In other words, less voltage wasted in the device and less heat generated in the switch itself.

So even though I moved away from a transistor-pair design, I did not lose the function of the original part. I kept the behavior that matters and improved the headroom of the switching element.

## How I made the turn-off behavior match the original

Injector drivers live and die by what happens **when they turn off**.

When the injector is energized, current builds through the coil. When the driver turns off, that current cannot just disappear instantly. The inductor pushes back and the voltage rises until the energy finds somewhere to go.

If you clamp that event too softly, the injector current decays slowly and the injector closes slower.

If you clamp it at a controlled higher voltage, the current decays faster and the injector shuts off faster.

That is why the original STA460C includes avalanche behavior, and that is why I made the clamp network on this board a priority.

The **SMCJ43A TVS** gives the injector energy a controlled place to go during turn-off. Instead of leaving the MOSFET to absorb everything uncontrolled, the external TVS defines the clamp behavior. The MOSFET itself is still avalanche-capable and rugged, but the design is not leaning on transistor breakdown alone as the primary strategy. 

This is one of the places where I believe the design really shines.

The original part had to integrate everything into one older transistor package.

I had the advantage of being able to break the problem into pieces and choose parts deliberately:

- one part for switching
- one part for gate protection
- one part for gate control
- one part for inductive clamping

That let me tune the behavior by function instead of being trapped inside the compromises of a single legacy device.

## Why the scope behavior is so similar to the STA460C

Even though this is not a Darlington design, the scope behavior under injector load is very similar to the original STA460C.

That is not an accident.

The reason the waveforms look so close is because both circuits are solving the same physical problem in nearly the same way at the system level.

During injector **on-time**, both circuits pull the low side of the injector toward ground and allow current to build through the coil.

During injector **turn-off**, both circuits allow the switching node to rise into a controlled clamp region so the magnetic field can collapse and the stored energy can be released safely.

That is why the scope captures show the same overall injector-driver signature:

- a low on-state region
- a sharp inductive kick at turn-off
- a controlled decay region
- a return toward baseline

So even though the internal device physics are different, the external circuit behavior is intentionally the same.

That is exactly what I wanted.

I was not trying to make a “MOSFET version” that behaved differently. I was trying to make a modern circuit that reproduced the **functional character** of the original injector driver.

The fact that the real injector-load scope captures line up so closely with the STA460C is strong evidence that the design approach worked.

## Where this design exceeds the original

There are a few areas where this design has clear advantages on paper.

The original STA460C is rated around **60 V** collector-emitter and **200 mJ** single-pulse avalanche energy. :contentReference[oaicite:6]{index=6}

The BSC025N08LS5 is rated **80 V** drain-source and **370 mJ** single-pulse avalanche energy, and it is explicitly listed as avalanche-tested. :contentReference[oaicite:7]{index=7}

The MOSFET also has far greater current and power capability than what this application normally demands. That does not mean the finished board should be treated like a giant current switch just because the silicon is beefy. The board layout, copper, connector system, and real-world ECU environment still matter. But it does mean the transistor itself has a lot more breathing room than the original legacy part. 

The low **RDS(on)** is another major improvement. Lower switching loss means less heat and less wasted energy in the device during injector on-time. :contentReference[oaicite:9]{index=9}

The externalized clamp strategy is also a strength. Instead of relying entirely on the internal avalanche behavior of an older Darlington device, this design uses a dedicated TVS to help shape and control the energy release event.

## Educational takeaway

One of the biggest things I want people to take from this project is that replacing an obsolete automotive part does not always mean cloning the exact internal transistor technology.

Sometimes the better approach is to ask:

**What job was the original part really doing?**

Once that is clear, you can rebuild that behavior using modern parts with more availability, more margin, and better understanding.

That is what I did here.

I did not just replace a transistor.

I replaced a **function**.

And by choosing the MOSFET, gate network, and clamp parts carefully, I was able to make the circuit behave like the original STA460C under real injector load while also gaining modern device advantages in voltage margin, avalanche robustness, and conduction efficiency.

## Final note

This project is part of a bigger idea behind ALPHA.

A lot of old Honda ECU hardware is still useful, still worth learning from, and still worth keeping alive. But too much of it is locked behind obsolete parts, bad documentation, or decades of gatekeeping.

I want these projects to do more than just hand someone a board file.

I want them to explain the circuit, teach the behavior, and make it easier for the next person to understand why it works.
