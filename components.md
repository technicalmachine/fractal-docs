# Components

Components contain code in one of several supported languages, and communicate with other components, whether or not they're in the same language or on the same processor. Using the defined component interfaces, a compiler can automatically generate APIs to access the functionality of a component from any supported language, or serialization code to access the components over USB or the network.

Components are the isolation boundary of the system. All mutable state is encapsulated in a component. Debug tools can analyze the ROM, RAM, and CPU consumption of each component to help developers target optimization efforts.

Components define **actions**, which combine object-oriented methods and events with a state machine. Data is passed into the component when an action begins, and passed out of the component when the action ends. Unlike methods in OOP, the beginning and end are explicit, separate occurrences, and the time between between entry and exit represents a **state** of the state machine. Nested actions may be defined within a parent action's state.

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

As an optimization, a sequence of the same action repeated several times can be submitted as a batch. All the *begin* data to be passed into the component must be available when the batch is submitted, and the *end* data is not available to the caller until the batch completes.

Components may implement batched actions specially (e.g using DMA), but if they don't, code will be generated to call the normal implementation repeatedly. When the target component is remote, this allows multiple actions without round trip latency.

### Asynchronous pipelined streaming

As a variant of batched actions, a stream of `begin` data can be sent to a device, and a stream of `end` data returned. This is mainly useful over a network, as code within one device would prefer to avoid buffering.

### Persistent parameters

Some parameters to an action are unlikely to change over subsequent calls. When a component declares its dependency on another component, it can provide upfront parameter values that will be used for the actions on that component. The target component can provide implementations that set parameter values persistently across multiple action invocations. These can e.g. write to a peripheral configuration register once, instead of on each action invocation, or send a value across the network once, instead of repeatedly.

If a persistent implementation is provided but the user does not preconfigure the parameter, code will be generated to update the persistent parameter on each event. If the parameter is set ahead of time but no persistent implementation is provided, it will automatically be passed on each event (which may still enable constant folding).

This is similar to run length encoding (for things backed by sampled signals), loop invariant code motion (for things backed by config registers) and currying (if you look at stacking components as function composition).

### YAML

Components are defined in .yaml files. It's probably useful to look at [an example](lib/example/spi.yaml) when reading this section.

The top level mapping specifies the implementation language and language-specific options:

```
backend: c
struct: spi_state
```

The top level mapping (the existence of the component itself) is itself an action and contains the action properties below.

In this section "this component" refers to the component being defined by this YAML file. "Dependency" refers to a component passed in as a top level argument, whose actions are used by this one. "Dependent" refers to a component stacked on top of this component, which uses its actions.

#### Actions

 * `args_in`: declares the input arguments passed into this action from the dependent when it begins.
 * `args_out`: declares the output arguments passed out of the action to the dependent on end. See "Arguments" below for the structure of these maps. Argument names may not be duplicated between `args_in` and `args_out`.


 * `on_begin`: a snippet of code run when a dependent begins this action. The code is run in a context with local variables for each argument listed in `args_in`.  

 * `to_begin`: the name of the auto-generated function or method that code in this component can call to begin this event and emit an event in the dependent.

 The choice of `on_begin` or `to_begin` defines whether this component begins the event, or the dependent does. Therefore, it is an error to specify both.

 * `on_end`: a snippet of code run when a dependent ends this action.  

 * `to_end`: the name of the auto-generated function or method that code in this component can can call to end this event and emit an event to the dependent. The generated function named in `to_end` accepts parameters for each argument listed in `args_out`.

 The choice of `on_end` or `to_end` defines whether this component ends the event, or the dependent does. Therefore, it is an error to specify both.

 * `actions`: declares the sub-actions of this action. The property is a name-value map, with the values having the structure defined in this section.

#### Arguments

An action's `args_in` and `args_out` contain one name-value pair for each argument. If the value is a string, it is interpreted as the name of a type as a shorthand for the full structure. Otherwise, it must be a map with the following properties:

 * `type`: The type of this argument. One of `int`, `real`, `symbol`, `byte`, or `component`. Component arguments are only allowed on the top-level action of the component so they can be initialized statically. (this may be relaxed in the future).

 * `actions`: For component arguments (defining dependencies), a map of sub-actions to be used by this component. Keys are action names as declared in the component used, and values are maps with the following keys:
  * `to_begin` / `on_begin` / `to_end` / `on_end`, with the same meaning as in the declaration of actions, but here defining the functions and callbacks for using the dependency. The `to` / `on` must be the opposite of the one used in the dependency's component definition because this calls/is called by the code in the dependency.

 * `args` is a map of values for persistent parameters.

 * `configure`: Code snippet executed when the persistent value of this argument changes (see "Persistent parameters" above)

 * `min`, `max`: For `real` and `int` arguments, the minimum and maximum acceptable values.

 * `values`: For `symbol` arguments, the list of valid symbols.

## Bindings

### Rust

  - optional stack switching or eventually [coroutines from generators](https://github.com/rust-lang/rust/issues/7746).
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