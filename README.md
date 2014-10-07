fractal-docs
============

A brief introduction to Fractal: Software Defined Hardware

## What is Fractal?

Fractal is a code generation platform for distributed embedded systems. Fractal's primary focus is breaking the IO into reusable chunks, called Components, with structured interfaces defined by state machines. Fractal automatically generates code for them to communicate with each other, whether they're on the same physical hardware or across a network. 

## Why is Fractal useful?

### Focus on Domain Logic, Not Configuration

Most of the code in an embedded system deals with IO, both with sensors and actuators, as well as communicating with a PC, phone, or server. Typically the domain logic implementing the unique behavior that the developer actually cares about is a small portion of the code, but ends up intertwined with all the IO, making it hard to introspect and port between platforms. By simplifying IO, developers can focus on the domain logic unique to their application and IO interfaces become standardized across chip families.

### Use The Right Language For The Job

Multiple programming languages can be mixed and matched, so developers can pick the right language for each task -- Rust and C for size and speed, JS or Lua for familiarity, ease of prototyping, and existing network libraries, and Signalspec for easy definition of protocol state machines. Prototype devices can run components of the design in higher level languages on a PC, transparently using hardware on the device. Similarly, hardware components may be replaced by software simulation for prototyping or testing.

### Debug Systems Painlessly

By automatically providing the framework for monitoring messages between the components, we can provide an unprecedented level of introspection for debugging to bring an experience like modern browsers' "Web Inspector" panel to embedded development. A hierarchical timeline could show event timing, CPU consumption, and the data involved at each level of abstraction.

## How Does Fractal Work?

Fractal breaks module subsystems into Components. Each Component can define a set of actions which it exposes to other components. In this way, bindings can be created automatically and each component is sandboxed into its own language and requirements.

Please read more about Components [here](https://docs.google.com/document/d/1QyKLxJx8jeOsswFgsxrbwtxuU1_REGMliJOWIkz5GB0/edit#) and please do leave comments.

