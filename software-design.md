As described above, the power of Fractal is derived from correlating specific code blocks with specific hardware chips.

There are several elements in Fractal's software development design:

* **Fractal Files** - YAML files that define the interface of each `Component` and include code that executes when those interfaces are accessed.
* **Interface Compiler** - The compiler that creates the bindings between different `Component`s and compiles the `Component`'s code for whichever target (Cortex M3, Cortex M0, POSIX, etc) is being used.
* **SignalSpec** - An optional, new language for concisely defining the transformation of data in two directions. Used primarily for sending and receiving data over a communication bus.

### Fractal Files

Each component is defined in a YAML file where each of the available actions is outlined defined and implemented in a single language. You can see a more complete documentation of `Component` specifications are [here](components.md). 

[Here is an example of a Fractal SPI controller](examples/spi.yaml) on an LPC18xx microcontroller written in C. You'll notice that each element in its interface is an `action` and C code is written for each of the standard `Component` functions (`to_begin`, `to_end`, `configure`, etc.) as described in the [`Component` documentation](component.md).

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

In this case, the generic accelerometer driver or the tap detector could be placed on top of any microcontroller with an I2C interface an **no code would have to be changed.** Additionally, the language each of the `Components` are implemented is inconsequential because a standard, statically-checked interface is exposed by each of them.