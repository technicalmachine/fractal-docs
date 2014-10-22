# Fractal

**Note**: Fractal is still under development and, while our goals and problem areas are well established, our methodology has room to change. Feedback is very welcome! team@technical.io. 

###What is Fractal?

Fractal helps you prototype a hardware device and its firmware, then iterate towards a design that can be produced at scale. It breaks software and hardware into reusable components, so you can focus on the novel aspects of your design, and leverage shared building blocks for the "boring" pieces like drivers, reference designs, and part footprints in an integrated manner.

Start prototyping by assembling modules with our development boards, and you can read sensor data and control actuators before writing any code. Prototype your device's behavior in your favorite high-level language, but easily drop into lower-level code where you need to be closer to the hardware. Debug with unprecedented introspection into the interaction between components in the system.

As you move toward production, optimize your design with transparency into microcontroller resource consumption, parts costs, and sources. Combine reference schematics and layouts to move from development boards to your own hardware.

### The Process to from Hardware Prototype to Scale

**Fractal** will enable engineers to follow a smooth transition from the prototyping phase of an an embedded device to the manufacturing phase. The objectives of those two phases are generally quite different and can be optimized by using different tools. 

In the prototyping phase, the priority is *time* : to prove if and how a device *should* be made as quickly as possible so that time and money aren't spent building something people don't want. We've found that using high level languages and flexible hardware platforms (Tessel, Arduino, RasPi, etc.) offer the fastest path to completing this phase.

In the manufacturing phase, the priority is *cost*. The goal is to figure out the cheapest and most efficient way to build a device in quantity because small deviations in cost at the unit level are magnified in bulk. This phase is really about optimization: making code footprints small so cheaper microcontrollers can be used, finding "sourcable" parts, and using the simplest sensors and actuators that can accomplish the same tasks as the prototype.

These competing goals often make it difficult to transition from a rough prototype to a scalable product without starting from scratch, with both software and hardware, at different stages. The goal of Fractal is to eliminate those rewrites while still preserving the benefit of using the best tool for each phase.


### The Software Benefits of Tessel

#### Focus on Domain Logic, Not Configuration

Most of the code in an embedded system deals with IO, both with sensors and actuators, as well as communicating with a PC, phone, or server. Typically the **domain logic implementing the unique behavior that the developer actually cares about is a small portion of the code**, but ends up intertwined with all the IO, making it hard to introspect and port between platforms. By simplifying IO, developers can focus on the domain logic unique to their application and **IO interfaces become standardized across chip families**.

#### Use The Right Language For The Job

**Multiple programming languages can be mixed and matched, so developers can pick the right language for each task** -- Rust and C for size and speed, JS or Lua for familiarity, ease of prototyping, and existing network libraries, and Signalspec for easy definition of protocol state machines. Prototype devices can run *Components* of the design in higher level languages on a PC, transparently using hardware on the device. Similarly, **hardware components may be replaced by software simulation** for prototyping or testing.

#### Debug Systems Painlessly

By automatically providing the framework for monitoring messages between the *Components*, we can provide **an unprecedented level of introspection for debugging to bring an experience like modern browsers' "Web Inspector" panel to embedded development**. A hierarchical timeline could show event timing, CPU consumption, and the data involved at each level of abstraction which could also help pare down hardware capabilities when picking cheaper components.

### The Hardware Benefits of Fractal

#### Get Part Suggestions
We would like to use the metadata of a program to suggest alternate parts that may be cheaper, easier to source, or more efficient than what's currently in use. 

#### Schematic and PCB Generation
If the parts data, extracted from the firmware files, are known, generating part models, schematics, and PCBs becomes an extremely complicated problem of datasheet parsing.

#### Manufacturing
Once gerber files (exported from the PCB design) are available, Fractal will search for the best of known PCB manufacturers and assembly houses to find the best candidate for the job. This is currently a very hands-on, relationship-based process but we'd like to eventually automate it (much like travel agents have been made largely obselete). This is very, very far off.


### So how does this complex, naive, never-gonna-work project actually do those things?

As described above, the power of Fractal is derived from correlating specific code blocks with specific hardware chips.

There are several elements in Fractal's software development design:

* **Fractal Files** - YAML files that define the interface of each `Component` and include code that executes when those interfaces are accessed.
* **Interface Compiler** - The compiler that creates the bindings between different `Component`s and compiles the `Component`'s code for whichever target (Cortex M3, Cortex M0, POSIX, etc) is being used.
* **SignalSpec** - An optional, new language for concisely defining the transformation of data in two directions. Used primarily for sending and receiving data over a communication bus.

### Fractal Files

Each component is defined in a YAML file where each of the available actions is outlined defined and implemented in a single language. You can see a more complete documentation of `Component` specifications are [here](components.md). 

[Here is an example of a Fractal SPI controller](examples/spi.yaml) on an LPC18xx microcontroller written in C. You'll notice that each element in its interface is an `action` and C code is written for each of the standard `Component` functions (`to_begin`, `to_end`, `configure`, etc.) as described in the [`Component` documentation](components.md).

This file will let us define the same interface for all SPI peripherals (a single `transaction`) across different microcontroller families. The only thing that has to change is the actual C implementation of which registers are called.

### Interface Compiler (this is where diagrams would be extra helpful)

By associating a series of these fractal files, you could make up an entire microcontroller:

```
lpc18xx :

- spi-lpc18xx.fractal
- i2c-lpc18xx.fractal
- uart-lpc18xx.fractal
- spifi-lpc18xx.fractal
- sct-lpc18xx.fractal
- dma-lpc18xx.fractal
- gpio-lpc18xx.fractal
```

Building on top of these abstractions can be easily imagined as literally stacking these `Components`. This is what building a JavaScript-based temperature driver on top of the i2c driver could look like:

[ tap-detector.c ]  

[ accelerometer.js ] 

[ i2c-lpc18xx.fractal]  

In this case, the generic accelerometer driver or the tap detector could be placed on top of any microcontroller with an I2C interface an **no code would have to be changed.** Additionally, the language each of the `Components` are implemented is inconsequantial because a standard, non-dynamic interface is exposed by each of them.
