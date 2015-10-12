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

In any block that is longer than one line, drop the first line down and continue from that indentation. Do the same for the accompanying block, even if it is short. This may seem overkill at first, but it really helps keeping the code clean as the project progresses.

**Good**

```elm
transformN : Int -> (a -> b) -> List a -> List b
transformN count transform list =
  let
    transformed =
      list
        |> List.take count
        |> List.map transform
    rest =
      list
        |> List.drop count
  in
    transformed ++ rest

ifElseExample : Int -> List a -> List a
ifElseExample threshold list =
  if List.length list > threshold
    then
      list
        |> doThings
        |> someMore
    else
      list
```

**Bad**

```elm
doThings this that =
  let mishymushy = mix this that
      mushymishy = mix that this
  in  [ mishymushy, mushymishy ]

ifElseExample threshold list =
  if List.length list > threshold
    then
      list
        |> doThings
        |> someMore
    else list
```

Here the bad style sacrifices code maintainability in the name of less lines of code. Note that the `let-in` guideline is different from the official Elm style guide. This is because in my opinion reordering the `let` block contents should not require moving the keyword from one line to another. If you were to change the order of `mishymushy` and `mushymishy` definitions, it would end up looking quite messy in version control diffs.

In the `ifElseExample` the only difference in the good and bad examples is the `else` block. For visual continuation, an indented else block is preferred.


## Controls

### `if-else`

Add a newline after the `if` clause and have `then` and `else` indented under it. As mentioned above, unless both the blocks are very short, it's good practice to drop the block contents down for both of them.

Don't play with lines just to make the block a one-liner.

**Good**

```elm
if needleInHaystack
  then actAccordingly
  else doNothing

if needleInHaystack
  then
    haystack
      |> transform
      |> filterRelevant
  else
    haystack
```

**Bad**

```elm
if needleInHaystack then actAccordingly else doNothing

if needleInHaystack
  then haystack |> transform |> filterRelevant
  else haystack
```

### `case-of`

- Add a newline after the `case-of` expression and each case's `->`
- **Always** have a general case at the end to prevent possible future fall-throughs

**Good**

```elm
valueOf : Thing -> Value
valueOf thing =
  case thing of
    Diamond diamond ->
      wow diamond
    PocketLint ->
      oh
    _ ->
      iDontEvenKnow
```

**Bad**

```elm
value = case thing of
  Diamond diamond  -> wow diamond
  PocketLint       -> oh
```

The `case thing of` clause should be one line down for glanceability.

"Rhythmic" indentations for the `->` arrows may look nice, but they quickly become a nightmare for maintainability. Simply adding a parameter to any of the cases can push the arrow on that line further than the others, requiring modifications on all lines.

Only covering current cases in `case-of` structures is very dangerous and should be avoided at all costs. As of elm-compiler 0.15.1 at least, the compiler cannot find possibilities for case fall-through. This means **adding a new possible `Thing` could cause runtime exceptions** without the default `_` case!

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
- Prefer `|>` (each on new line), starting with whatever feels most natural
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
updateCar : Action -> Car -> Car
updateCar action car =
  case action of
    Refuel -> { car | fuelPercentage <- 100 }
    Drive distance ->
      { car | fuelPercentage <- car.fuelPercentage - (getConsumption distance car), odometer <- car.odometer + distance }

mtl m = withDefault [] (map (repeat 1) m)
```

While very compact, there are multiple things that make this code worse than the above:

- `updateCar` has a way too long line that could easily be split.
- The `case-of` can fall through if a new `Action` gets added in the codebase.
- Using imports with `exposing (..)` pollutes the namespace and makes it impossible to know which module is responsible for the functions `map` and `repeat` for example.
- Missing type annotation makes matters even worse.
- Indescriptive names cause mental overhead.
- No newline after equals sign will lead to 1) long lines and 2) worse version control diffs.
- Parens syntax is harder to glance through than `|>`.


#### Trickier parts




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

