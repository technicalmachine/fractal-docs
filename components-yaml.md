### Component definition YAML

Components are defined in [.yaml](https://en.wikipedia.org/wiki/YAML) files. It's probably useful to look at [an example](lib/examples/spi.yaml) when reading this section.

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
