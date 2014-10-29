# Fractal Software

Fractal splits the firmware into Components, which contain code in one of several supported languages. Using the defined component interfaces, a compiler can automatically generate APIs to access the functionality of a component from any supported language, or serialization code to access the components over USB or the network.

Components are the isolation boundary of the system. All mutable state is encapsulated in a Component. Debug tools can analyze the ROM, RAM, and CPU consumption of each component to help developers target optimization efforts.

Code in existing languages, including C, Rust, JS, and Lua can be bound into Components with [**YAML files**](components-yaml.md) that define the interface of the Component and include code that executes when those interfaces are invoked.

You can optionally use [**Signalspec**](http://signalspec.org) to implement IC-specific protocols on top of SPI, I2C, UART interfaces to communicate with other ICs on your board. Signalspec is a domain-specific language for modeling protocols using regex-like descriptions of state machines.

These languages can be mixed and matched to use the best tool for each part of the application. You might use a C Component for LPC1830 I2C, and and above that use a Signalspec description of the MMA84 accelerometer protocol, and then a JS component on top that POSTs to a HTTP endpoint when it detects freefall. When going to production, porting your prototype's JS domain logic into Rust to fit on your Cortex M0 is much simpler than starting the firmware from scratch.

The **Interface Compiler** compiles each of the Components and generates binding glue code to connect them. It then adds lightweight runtime support code and links an executable for whichever target (Cortex M3, Cortex M0, Embedded Linux, etc.) is being used.

## Design goals

 * Do whatever you can at compile time (error checking, resource allocation, and optimization)
 * Features you don't use should have zero runtime cost
 * Scale from Cortex M0 to Core i7
 * Re-use familiar technologies

## Component Actions

A Component's interface is defined by **actions**, which combine object-oriented methods and events with a state machine. Data is passed into the component when an action begins, and passed out of the Component when the action ends. Unlike methods in OOP, the beginning and end are explicit, separate occurrences, and the time between between entry and exit represents a **state** of the state machine.

Actions are nested hierarchically, somewhat like [Harel / UML Statecharts](http://www.wisdom.weizmann.ac.il/~harel/SCANNED.PAPERS/Statecharts.pdf). For example, SPI bytes can only occur within a transaction, and transactions can only occur within an instance of the SPI component (whose existence behaves like an action, because it can have a start and an end, and has actions within it).

The timing of an action's beginning and end can be determined either by the component defining it ("component") or the component using it ("user"):

  - user begins, user ends: mode or output state
    - SPI master transaction
    - GPIO output
    - Interrupt enabled
  - user begins, component ends: asynchronous completion
    - SPI byte
    - I2C master byte
    - UART TX byte
  - implicit begin, component ends: asynchronous event
    - pin change interrupt
    - UART RX byte
  - component begins, user ends
    - I2C slave byte with clock stretching
  - component begins, component ends:
    - level interrupt / GPIO input, kind of
    - SPI slave transaction

Action parameters are one of the following types
 - symbol
 - integer (min, max)
 - real (min, max, SI units)
 - byte

Note that there are no dynamically sized types, so all memory can be allocated statically. Strings and buffers are represented by a series of actions that each transfer a byte or sample. (But see "Batched actions" below)

### Batched actions

As an optimization, a sequence of the same action repeated several times can be submitted as a batch. All the *begin* data to be passed into the Component must be available when the batch is submitted, and the *end* data is not available to the caller until the batch completes.

Components may implement batched actions specially (e.g using DMA), but if they don't, code will be generated to call the normal implementation repeatedly. When the target Component is remote, this allows multiple actions without round trip latency.

### Asynchronous pipelined streaming

As a variant of batched actions, a stream of `begin` data can be sent to a device, and a stream of `end` data returned. This is mainly useful over a network, as code within one device would prefer to avoid buffering.

### Persistent parameters

Some parameters to an action are unlikely to change over subsequent calls. When a Component declares its dependency on another component, it can provide upfront parameter values that will be used for the actions on that component. The target component can provide implementations that set parameter values persistently across multiple action invocations. These can e.g. write to a peripheral configuration register once, instead of on each action invocation, or send a value across the network once, instead of repeatedly.

If a persistent implementation is provided but the user does not preconfigure the parameter, code will be generated to update the persistent parameter on each event. If the parameter is set ahead of time but no persistent implementation is provided, it will automatically be passed on each event (which may still enable constant folding).

This is similar to run length encoding (for things backed by sampled signals), loop invariant code motion (for things backed by config registers) and currying (if you look at stacking components as function composition).

## Language bindings

Each component is defined in a YAML file where each of the available actions is outlined, defined, and implemented as [documented here](components-yaml.md).

[Here is an example of a Fractal binding](examples/spi.yaml) for the SPI peripheral on a typical microcontroller written in C. You'll notice that each element in its interface is an `action` and C code is written for each of the standard Component functions (`to_begin`, `to_end`, `configure`, etc.) as described in the [`Component` documentation](components-yaml.md).

This file will let us define the same interface for all SPI peripherals (a `transaction`, that contains a series of bytes) across different microcontroller families. The only thing that has to change is the actual C implementation for the registers and interrupts that vary from vendor to vendor.

### Rust

  - optional stack switching or eventually [coroutines from generators](https://github.com/rust-lang/rfcs/issues/388).
  - state struct

### JS
  - Isolated, bounded heap
  - Actions can be mapped to the most appropriate JS interface:
      - Methods with callbacks
      - Promises
      - EventEmitters
      - Streams

### Lua
?

### C

  - optional stack switching or [protothreads](http://dunkels.com/adam/pt/)
