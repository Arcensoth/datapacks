# Function Documentation Format

> This document is a **work-in-progress**!

- [Function headers][function-headers]
- [Function annotations][function-annotations]
  - [`@user`][function-annotation-user]
  - [`@public`][function-annotation-public]
  - [`@context`][function-annotation-context]
  - [`@within`][function-annotation-within]
  - [`@handles`][function-annotation-handles]
  - [`@args`][function-annotation-args]
  - [`@args_nbt`][function-annotation-args_nbt]
  - [`@returns`][function-annotation-returns]
  - [`@returns_nbt`][function-annotation-returns_nbt]

The goal is to create a standard function documentation format that:

1. is easy to read and write; and
2. provides useful information to the developer; and
3. allows external tools (such as syntax highlighters and documentation compilers) to quickly and reliably identify the different components of a function.

## Function headers

A function always begins with a series of comments that describe its purpose, context, and interface:

```mcfunction
#> imp:core/load
#
# Main loading procedure with the core entity as context.
#
# Anything branching from here can use `@s` instead of the core UUID.
#
# @context core
```

> Function headers do not contain information such as: who created the function, when it was last updated, etc. This information is expected to be handled automatically by a version control system such as Git. Contributors to the datapack should be listed in a centralized location easily accessible to the end-user, such as the datapack description and/or a user function that can be run in-game.

The first line of the function header is preceded by `>` and contains nothing but the function name:

```mcfunction
#> imp:core/load
```

This line requires adjustment when changing the name/location of the function, but makes it clear which function is being modified and easy to copy-paste into a `function` command.

> The first line of the function header is preceded by `>` to denote an imaginary block comment. This is an alternate, vanilla-compatible comment style that allows syntax highlighters and other external tools to quickly and reliably identify block comments.

The next section of the function header is description of the function's purpose. This can be split into multiple paragraphs:

```mcfunction
# Main loading procedure with the core entity as context.
#
# Anything branching from here can use `@s` instead of the core UUID.
```

Following the description is a series of annotation tags that define the function's interface:

```mcfunction
# @context core
```

Similar to the reason why we use the block comment syntax, these annotations help external tools identify distinct components of the function. See the section on [annotations] for details regarding the purpose and capabilities of each individual annotation.

## Function annotations

### Function annotation `@user`

Declare that the function is a _user_ function. User functions are intended to be run from chat by operators.

```mcfunction
#> imp:-user/menu
#
# @user
```

### Function annotation `@public`

Declare that the function is a _public_ function. Public functions are designed to be called by other modules, and as such should be well-documented and tested for other developers.

```mcfunction
#> imp:utils/resolve_text
#
# Resolve a text component.
#
# @public
```

### Function annotation `@context`

Declare the entity execution context that is expected at runtime. Any context outside what is declared may lead to erroneous or undefined behaviour.

The context is an arbitrary string but should be minimally descriptive. For example, `root` is generally accepted to mean the server root context:

```mcfunction
#> imp:load
#
# Root entry point for the main loading procedure.
#
# @context root
```

The context need not be self-explanatory, but it is good practice to be unambiguous and consistent:

```mcfunction
#> imp:core/load
#
# Main loading procedure with the core entity as context.
#
# Anything branching from here can use `@s` instead of the core UUID.
#
# @context core
```

Sometimes it's necessary to be more verbose, especially if the context is being used in a very limited scope:

```mcfunction
#> imp:log/calc/coords/inner
#
# @context temporary entity used to extract coords from nbt
#
# @within imp:log/calc/coords
```

Public functions commonly use `@context any` which does not have any special meaning but serves to compliment `@public` by explicitly stating that the function will operate correctly under any context:

```mcfunction
#> imp:utils/resolve_text
#
# Resolve a text component.
#
# @public
#
# @context any
```

### Function annotation `@within`

Declare that the function is a **child** function and should only ever be called from _within_ one of its **parent** functions.

This pattern is commonly used when a function needs conditional logic but it would be suboptimal to include said logic in the same function. This is where a branch is used, and how function trees are constructed.

> Function trees are generally constructed from several layers of parent and child functions. It is unusual to call a function tree from somewhere in the middle, unless the caller is well-aware of how it is constructed and any assumptions that need to be fulfilled.

This annotation has two forms. The first is a short-form that should be used when there is only one parent function:

```mcfunction
#> namespace:path/to/function
#
# An imaginary example function.
#
# @within namespace:some/parent/function
```

The second form spans multiple lines and can be used for any number of parent functions:

```mcfunction
#> namespace:path/to/function
#
# An imaginary example function.
#
# @within
#   namespace:some/parent/function
#   namespace:another/parent/function
```

This pattern may also be used for API functions that require a certain context, and would otherwise place the burder of setting the context on the developer. This also helps to ensure that future versions of the function remain backwards-compatible.

For example, a public parent function is created that calls the child function with the correct context:

```mcfunction
#> namespace:my/parent/function
#
# An imaginary parent function.
#
# @public
#
# @context any

execute as @e[type=minecraft:area_effect_cloud, tag=namespace.marker] at @s run function namespace:my/child/function
```

And the child function documents this using `@within`:

```mcfunction
#> namespace:my/child/function
#
# @context marker entity
#
# @within imp:utils/resolve_text
```

### Function annotation `@handles`

Declare that the function is an event handler for one or more function tags.

> Whether this should be used for all function tags is undecided, but the likely answer is: no. This annotation was primarily introduced to compliment `@context` in helping to remind the developer - at a glance - of the circumstances in which the function will be running.

Similar to `@within`, this annotation has two forms. The first form is used for a single event handler:

```mcfunction
#> imp:load
#
# Root entry point for the main loading procedure.
#
# @handles #minecraft:load
```

The second form is used for multiple event handlers:

```mcfunction
#> namespace:path/to/function
#
# An imaginary example function.
#
# @handles
#   #namespace:some/function/tag
#   #namespace:another/function/tag
```

### Function annotation `@args`

Declare arguments that should be present in the scoreboard before the function is called.

TODO document @args

```mcfunction
#> imp:utils/timestring
#
# Format a human-readable time string based on a number of ticks.
#
# [...]
#
# @args
#   $imp.utils.timestring.ticks __args__
#       The number of ticks to format.
```

### Function annotation `@args_nbt`

Declare arguments that should be present in the global NBT register before the function is called.

TODO document @args_nbt

```mcfunction
#> imp:utils/resolve_text
#
# Resolve a text component.
#
# [...]
#
# @args_nbt
#   __args__.imp.utils.resolve_text: raw_text_component
#       The raw/stringified text component to resolve.
```

### Function annotation `@returns`

Declare values that will be present in the scoreboard after the function completes.

TODO document @returns

```mcfunction
#> imp:utils/timestring
#
# Format a human-readable time string based on a number of ticks.
#
# [...]
#
# @returns
#   $imp.utils.timestring.days __return__
#       The total number of tick-days.
# [...]
```

### Function annotation `@returns_nbt`

Declare values that will be present in the global NBT register after the function completes.

TODO document @returns_nbt

```mcfunction
#> imp:utils/timestring
#
# Format a human-readable time string based on a number of ticks.
#
# [...]
#
# @returns_nbt
#   __return__.imp.utils.timestring.resolved: raw_text_component
#       The resolved time string.
```

[function-headers]: #function-headers
[function-annotations]: #function-annotations
[function-annotation-user]: #function-annotation-user
[function-annotation-public]: #function-annotation-public
[function-annotation-context]: #function-annotation-context
[function-annotation-within]: #function-annotation-within
[function-annotation-handles]: #function-annotation-handles
[function-annotation-args]: #function-annotation-args
[function-annotation-args_nbt]: #function-annotation-args_nbt
[function-annotation-returns]: #function-annotation-returns
[function-annotation-returns_nbt]: #function-annotation-returns_nbt
