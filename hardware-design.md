# Fractal - hardware considerations

## Scope/applicability

Initially, Fractal will target what we're loosely calling "industrial" applications and small distributed sensor systems (classical IoT problems), as opposed to "consumer" applications. In essence, this frees us from worrying about the specifics of the final form factor and eliminates industrial design, plastics, and ergonomics/human factors from the equation.

## "Design" vs. "generate"

Fractal aims to abstract away the tedious bits of the hardware design process. As such, perhaps "generation" is a better word here.

In a typical workflow, the user would specify the high-level domain logic of the system being designed and Fractal will generate not only the hardware that's appropriate for the task (a tiny, low-power microcontroller and radio for a wireless sensor vs. something capable of running a fast control loop for an actuator), but also help draw and generate interface code for the corresponding physical divisions within the system. For example, the tool might decide that the fusion of data streams collected on a single wireless probe should happen on the recieving side, and would generate a hardware design that is conducive to this system architecture.

##  Implementation

Much of this is still up in the air, but we've learned a lot from our work on [Tessel](www.tessel.io).

### Design files

Plaintext files, systematic net naming, and hierarchical schematic design makes the integration half of this relatively straightforward.

A "preprocessor" would combine the sub-schematics into a "final" master design. Similarly, a common interface (TBD, but we like LGAs more than the 1D, through-hole module interface Tessel uses right now) allows layout files, and even Gerbers, to be dropped into place as needed.

For now, initial design file generation would need to be done manually, but this is relatively easy with reference designs and could be automated with sufficient resource allocation.

### Interfaces and modularity

As stated earlier, the specifics are TBD here, but the gist is as follows:

* The vast majority of simple sensors and actuators can be controlled over one or two of SPI, I2C, UART, and GPIO
* The vast majority of systems currently within the scope of Fractal have at most 2,3 "subsystems" related to these devices
* Tessel's module interface (which includes all of the interfaces listed above and can be used as a proxy/analog for what is feasible with Fractal) has thus far been sufficient for pretty much anything we wanted to put into a module form factor.

Combined, this suggests that a common form factor of some kind, or perhaps a one for each of the interfaces listed above, is well within the realm of possibility. In keeping with the multiple but standard sizes of Tessel module (and geometric undertones of all our names), some experimentation with form factors is in order.

Also of note: some optimization for IC manufacturers/specific microcontroller families is likely in order here. Atmel, TI, Freescale, and NXP all do their peripherals a little differently.

### Power

At the risk of overgeneralizing, there are three buckets:

* External power of some kind is an option, be it line voltage (presumably through an AC adaptor), POE, or a power bus of some kind in a larger system
* The device must be battery powered
* The device uses very little power

The only reason I draw a distinction between the second and third categories is that primary cells + field service might make sense in some cases for the third category.

In all cases, after the initial "acquisition" is completed, most of the devices Fractal is well suited to will have a low power domain and may or may not have a high power domain. There exist many, many regulators (some of which are even pin-compatible) across input voltages, output voltages, and manufacturers that can allow the system to be tuned to fit the any requirements imposed by the environment.

## Open questions

This list is in no way exhaustive.

### Combination of processing and power

Many microcontrollers, by their nature, have power requirements that go hand in hand with their application/versatility. As such it may make sense to combine the two at the physical/Component level. 

### LGA for all vs. computation+power vs. base boards

As discussed above, it may make sense to combine processor and power plant. The next question is if we have a common footprint for everything as opposed to treating power and processor blocks as special.

The logical extension, given that logistics > plastic > silicon (in terms of cost), is whether the processor and power plant should be built into the substrate board or placed on their own "modules".

## Prototypical example as a reference

A prototypical system would include a microcontroller, radio, power supply, and sensor.

By offering a few pre-canned options for each functional block that can be interchanged (at least "on paper"), Fractal would immediately give the design team a working system, albeit one not tuned for the specific application. Different radios (any/or microcontrollers, depending on what the initial system design/spec specifies) could be swapped out in a matter of minutes, not days, thanks to the layers of software abstraction Fractal provides.

A different sensor (more precise, different range, different output type, etc.) within the Fractal ecosystem, or even something built in-house using provided templates, could be swapped out in a similar manner (and then added to the master Fractal library at the user's discretion, in the case of the in-house option).
