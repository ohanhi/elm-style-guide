# Elm Style Guide

Opinionated best practices for [Elm](http://elm-lang.org/) code style.

**Work in progress!**


There is an official [style guide](http://elm-lang.org/docs/style-guide) on the Elm website. They state:

> **Goal:** a consistent style that is easy to read and produces clean diffs. This means trading aggressively compact code for regularity and ease of modification.

I wholly agree with the sentiment here. However, the examples in the document are far from exhaustive. This is why I felt the need for a style guide of my own.

I base my opinions on the experience I've gained while:

- working in a customer project
- almost solely in Elm
- with two other developers
- over several months.

## Common style tips

- Line length <= 80
- Indentation 2 spaces
- No trailing spaces on lines
- Newline at end of file
- Write type annotations, records, case-ofs etc. with spaces between special characters. See examples for clarification.
- Use `elm-make --warn` and get rid of the warnings
  - Protip: for a completely fresh compilation, do `rm -rf elm-stuff/build-artifacts`

## Module definitions

- Import only needed modules
- Order of preference:
  1. non-exposing imports
  2. explicitly exposing imports
  3. imports exposing everything
- When feasible, explicitly define what to expose from current module

## Declarations

- Name functions descriptively
- Name parameters descriptively
- Write type annotations
- Prefer `|>`, starting with whatever feels most natural
- Avoid long functions
- Split long `let` blocks into separate functions
- Always add a newline after the `=` equal sign

**Good**

```elm
{- NB: function "getConsumption" is defined elsewhere in the module -}

updateCar : Action -> Car -> Car
updateCar action car =
  case action of
    Refuel ->
      { car | fuelPercentage <- 100 }    -- simple update
    Drive distance ->
      { car | fuelPercentage <- car.fuelPercentage - (getConsumption distance car)
            , odometer <- car.odometer + distance
            }
    _ ->  -- never have a possible case fall-through
      car

{- Generic type function example -}
maybeToList : Maybe a -> List a
maybeToList maybe =
  maybe
    |> Maybe.map (List.repeat 1)
    |> Maybe.withDefault []
```

**Bad**

```elm
mtl m = withDefault [] (map (repeat 1) m)
```

While very compact, there are multiple things that make this code worse than the above:

- Using imports with `exposing (..)` pollutes the namespace and makes it impossible to know which module is responsible for the functions `map` and `repeat` for example.
- Missing type annotation makes matters even worse.
- Indescriptive names cause mental overhead.
- No newline after equals sign will lead to 1) long lines and 2) worse version control diffs.
- Parens syntax is harder to glance through than `|>`.


## Types and records

- Don't be afraid of introducing type aliases for common things -- they help with type annotations and make refactoring far easier
- Avoid nested record declarations, define type aliases instead
- Use the Elm style
  1. opening brace on the first line
  2. leading commas on the same indentation
  3. closing brace on the same indentation

**Good**

```elm
type alias Car =
  { fuelPercentage : Float
  , odometer : Float
  , manufacturer : CarManufacturer
  , registrationDate : Date
  }

exampleCar =
  { fuelPercentage = 100
  , odometer = 0
  , manufacturer = volkswagen
  , registrationDate = Date.fromString "2015-10-12T07:33:23"
  }
```

**Bad**

```elm
type alias Car = {
  fuelPerc: Float,
  odo: Float,
  manuf: { name: String, prodVolume: Float }
  registrationDate: String
}

exampleCar = {
  fuelPerc = 100,
  odo = 0,
  manuf = { name = "Volkswagen", prodVolume: 3.13 },
  registrationDate = Date.fromString "2015-10-12T07:33:23"
}
```

While this syntax may feel more familiar coming from JavaScript, the above (Good) style formats the type alias declaration to be akin to function declarations.

Inline nested records can't be referenced anywhere, but the above `CarManufacturer` could be used in type annotations anywhere. Also, should the manufacturer record properties change, the good example will still work while the bad example will need refactoring.

The leading commas style with the braces aligned makes it glaringly obvious where the declaration starts and where it ends. It also avoids tiny syntax errors, like the one in the bad example. Did you notice it? No? How about the comma-revised version here:

```elm
type alias Car =
  { fuelPerc: Float
  , odo: Float
  , manuf: { name: String, prodVolume: Float }
  registrationDate: String
  }
```

Another plus for the style is that adding a line to the bottom will not alter any other lines. The obvious drawback is that you can't say the same for the first line. In my experience, it is *far more common* to add a new property towards the end and not as the very first line so this drawback I can live with.


## Working with `elm-html`

*TODO*

