# Fractal

**Note**: Fractal is still under development and, while our goals and problem areas are well established, our methodology has room to change. Feedback is very welcome! team@technical.io. 

###What is Fractal?

At the heart of **Fractal** is the idea that you can build a better product if the software and hardware are well integrated. **Fractal is a framework that uses the "metadata" of hardware and software designs to help inform the system architecture as whole**. By surfacing relevant resource consumption levels such as memory consumption, CPU utilization, and integrated circuit prices, the process to scaling down a hardware device becomes easier, more transparent, and shared amongst a team. 

For many embedded products, the *domain logic* and the final *Printed Circuit Board* are the primary deliverables of the electrical design process. **Fractal** aims to (eventually) fully automate, or at least help guide, any parts of the process that are not directly relevant to those deliverables such as writing bootup and part-specific drivers, picking parts and sources, and modeling parts and schematics. We value ownership over black boxes so we'd like ensure those intermediary steps are fully transparent and optimizable. Put ambitiously, we'd like to be able to generate a PCB from firmware. 

### The Process to from Hardware Prototype to Scale

**Fractal** will enable engineers to follow a smooth transition from the prototyping phase of an an embedded device to the manufacturing phase. The objectives of those two phases are generally quite different and can be optimized by using different tools. 

In the prototyping phase, the priority is *time* : to prove if and how a device *should* be made as quickly as possible so that time and money aren't spent building something people don't want. We've found that using high level languages and flexible hardware platforms (Tessel, Arduino, RasPi, etc.) offer the fastest path to completing this phase.

In the manufacturing phase, the priority is *cost*. The goal is to figure out the cheapest and most efficient way to build a device in quantity because small deviations in cost at the unit level are magnified in bulk. This phase is really about optimization: making code footprints small so cheaper microcontrollers can be used, finding "sourcable" parts, and using the simplest sensors and actuators that can accomplish the same tasks as the prototype.

These competing goals often make it difficult to transition from a rough prototype to a scalable product without starting from scratch, with both software and hardware, at different stages. The goal of Fractal is to eliminate those rewrites while still preserving the benefit of using the best tool for each phase.


### The Software Benefits of Fractal

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


### How will it work?

To read more about the architecture of the framework and how it works, continue reading about the [software design](software-design.md) and how it will interoperate with the [hardware design](hardware-design.md) of our prototyping tools. 


