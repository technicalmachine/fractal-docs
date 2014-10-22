# Fractal - hardware considerations

## Scope/applicability

Initially, Fractal will target what we're loosely calling "industrial" applications and small distributed sensor systems (classical IoT problems), as opposed to "consumer" applications. In essence, this frees us from worrying about the specifics of the final form factor and eliminates industrial design, plastics, and ergonomics/human factors from the equation.

Fractal's "EDA" output (schematics and layout files) can, from a seasoned EE's perspective, be seen as something that is good enough to deploy at scale, but not that something that considers the aforementioned requirements of consumer electronics.

## Prototypical example as a reference

A prototypical system would include a microcontroller, radio, power supply, and sensor.

By offering a few pre-canned options for each functional block that can be interchanged (at least in theory), Fractal would immediately give the design team a working system, albeit one not tuned for the specific application. Different radios (and/or microcontrollers, depending on what the initial system design/spec specifies) could be swapped out in a matter of minutes, not days, thanks to the layers of software abstraction Fractal provides.

A different sensor (more precise, different range, different output type, etc.) within the Fractal ecosystem, or even something built in-house using provided templates, could be swapped out in a similar manner (and then added to the master Fractal library at the user's discretion, in the case of the in-house option).

## "Design" vs. "generate"

Fractal aims to abstract away the tedious bits of the hardware design process. As such, perhaps "generation" is a better word here.

In a typical workflow, the user would specify the high-level domain logic of the system being designed and Fractal would generate not only hardware appropriate for the task (a tiny, low-power microcontroller and radio for a wireless sensor vs. something capable of running a fast control loop for an actuator), but also help draw and generate interface code for the resulting/corresponding physical divisions within the system.

##  Implementation

Much of this is still up in the air, but we've learned a lot from our work on [Tessel](www.tessel.io).

### Design files

Plaintext files, systematic net naming, and hierarchical schematic design makes the integration half of this relatively straightforward.

A "preprocessor" would combine the sub-schematics into a "final" master design. Similarly, a common interface (TBD, but we like SMT QFN/LGAs more than the 1D, through-hole module interface Tessel uses right now for their ease of assembly, what they allow for in PCB routing, and low cost) allows layout files, and even Gerbers, to be dropped into place as needed.

The layout manifestations of the `Components` need only resemble a physical QFN/LGA in that the interface is common across the `Components`; making the actual hardware available only in any given form factor would serve only to save *us* NRE costs and gains the end user nothing. In fact, we suspect that the desire for a "full-custom" PCB, as opposed to one that is all board on board, would justify the added cost for the end user.

For now, initial design file generation would need to be done manually, but this is relatively easy with reference designs and could be automated with sufficient resource allocation.

### Interfaces and modularity

As stated earlier, the specifics are TBD here, but the gist is as follows:

* The vast majority of simple sensors and actuators can be controlled over one or two of SPI, I2C, UART, and GPIO
* The vast majority of systems currently within the scope of Fractal have at most 2,3 "subsystems" related to these devices
* Tessel's module interface (which includes all of the interfaces listed above and can be used as a proxy/analog for what is feasible with Fractal) has thus far been sufficient for pretty much anything we wanted to put into a module form factor. It is, however, through hole and clunky: through-hole COTS plastics are large, expensive to purchase and install (vs. no connectors or SMT connectors), and therefore ill-suited to deployment.

Combined, this suggests that a common form factor of some kind, or perhaps one for each of the interfaces listed above, is well within the realm of possibility. In keeping with the multiple but standard sizes of Tessel module (and geometric undertones of all our names), some experimentation with form factors is in order.

Also of note: some consideration of IC manufacturers/specific microcontroller families is in order here. Atmel, TI, Freescale, and NXP all do their peripherals a little differently, so the interface we use must allow for flexibility in selecting a microcontroller, lest we inadvertantly lock ourselves into one manufacturer/family.

### Power

At the risk of overgeneralizing, there are three buckets:

 * Grid power of some kind (be it line voltage, an AC adaptor, POE, or a shared power bus in a larger system)
 * A rechargable battery that lasts hours to weeks
 * A non-rechargable battery that lasts months to years

The only reason I draw a distinction between the second and third categories is that primary cells + field service might make sense in some applications.

In all cases, after the initial "acquisition" is completed, most of the devices Fractal is well suited to will have a low power domain and may or may not have a high power domain. There exist many, many regulators (some of which are even pin-compatible) across input voltages, output voltages, and manufacturers that can allow the system to be tuned to fit any requirements imposed by the environment.

## Open questions

This list is in no way exhaustive.

### Combination of processing and power

Many microcontrollers, by their nature, have power requirements that go hand in hand with their application/versatility. As such it may make sense to combine the two at the physical/`Component` level. 

### LGA for all vs. computation+power vs. base boards

As discussed above, it may make sense to combine processor and power plant. The next question is if we have a common footprint for everything as opposed to treating power and processor blocks as special.

The logical extension, given that logistics > plastic > silicon (in terms of cost), is whether the processor and power plant should be built into the substrate board or placed on their own "modules".

### Hardware design team's interaction with Technical Machine

What is our role in this once the tool is out there and useful? How do we interact with customers and why do they interact with us? Do we serve as a conduit somewhere, and if so, how? etc.
