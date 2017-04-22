# Elm Style Guide

Opinionated best practices for [Elm](http://elm-lang.org/) code style.

**Note** A whole bunch of this guide is nowadays covered by [elm-format](https://github.com/avh4/elm-format). Definitely use that!

**Work in progress!**


There is an official [style guide](http://elm-lang.org/docs/style-guide) on the Elm website. They state:

> **Goal:** a consistent style that is easy to read and produces clean diffs. This means trading aggressively compact code for regularity and ease of modification.

I wholly agree with the sentiment here. However, the examples in the document are far from exhaustive. This is why I felt the need for a style guide of my own.

I base my opinions on the experience I've gained while:

- working in customer projects
- almost solely in Elm
- with multiple other developers
- over a couple of years.

### Common style tips

- Line length <= 80
- Indentation 4 spaces
- No trailing spaces on lines
- Newline at end of file
- Write type annotations
- Write records, `case-of`s etc. with spaces between special characters (`=`, `:`, `->`, etc).
- Use `elm-make --warn` and get rid of the warnings
  - Protip: for a completely fresh compilation, do `rm -rf elm-stuff/build-artifacts`

In any block that is longer than one line, drop the first line down and continue from that indentation. Do the same for the accompanying block, even if it is short. This may seem overkill at first, but it really helps keeping the code clean as the project progresses.


## Types, records and lists


### Type aliases

- Don't be afraid of introducing type aliases for common things -- they help with type annotations and make refactoring far easier
- Use the Elm style: type aliases are akin to function declarations.

```elm
-- ✅ GOOD
type alias Car =
    { fuelPercentage : Float
    , odometer : Float
    }

-- ❌ BAD
type alias BadCar = {
    fuelPercentage: Float,
    odometer: Float
  }
```


Avoid nested record declarations, define type aliases instead

```elm
-- ✅ GOOD
type alias CarManufacturer =
    { name : String
    , prodVolume : Float
    }

type alias CarMeta =
    { manufacturer : CarManufacturer
    -- other fields
    }

-- ❌ BAD
type alias BadCarMeta =
    { manufacturer: { name: String, prodVolume: Float }
    -- other fields
    }
```

Inline nested records can't be referenced, but the above `CarManufacturer` could be used in type annotations anywhere. Also, should the manufacturer record properties change, the good example will still work while the bad example will need refactoring.


### Records

```elm
-- ✅ GOOD
goodCar =
    { fuelPercentage = 100
    , odometer = 0
    }

-- ❌ BAD
badCar = {
  fuelPerc = 100,
  odo = 0
}
```

### Lists

```elm
-- ✅ GOOD
carBrands =
    [ "Aston Martin"
    , "Audi"
    , "BMW"
    , "Buick"
    ]

-- ❌ BAD
badCarBrands = ["Aston Martin", "Audi", "BMW", "Buick"]

badCarBrands' = [
    "Aston Martin",
    "Audi",
    "BMW",
    "Buick"
    ]
```

The leading commas style with the braces aligned makes it glaringly obvious where the declaration starts and where it ends.

Another plus for the style is that adding a line to the bottom will not alter any other lines. The obvious drawback is that you can't say the same for the first line. In my experience, it is *far more common* to add a new property towards the end and not as the very first line so this drawback I can live with.




### `let-in`

```elm
-- ✅ GOOD
doThings this that =
    let
        mishymushy =
            mix this that
                |> andDoStuff

        mushymishy =
            mix that this
    in
        [ mishymushy
        , mushymishy
        ]

-- ❌ BAD
badDoThings this that =
  let mishymushy = andDoStuff (mix this that)
      mushymishy = mix that this
  in  [ mishymushy, mushymishy ]
```

Here the bad style sacrifices code maintainability in the name of less lines of code. Note that the `let-in` guideline is different from the official Elm style guide. This is because in my opinion reordering the `let` block contents should not require moving the keyword from one line to another. If you were to change the order of `mishymushy` and `mushymishy` definitions, it would end up looking quite messy in version control diffs.


### `if-else`

Add a newline after the `if expression then` and `else` indented to the same level. Always drop the block contents down for both of the branches.

```elm
-- ✅ GOOD
if needleInHaystack then
    actAccordingly
else
    doNothing

-- ❌ BAD
if needleInHaystack then actAccordingly else doNothing


-- ✅ GOOD
if needleInHaystack then
    haystack
        |> transform
        |> filterRelevant
else
    haystack

-- ❌ BAD
if needleInHaystack
    then haystack |> transform |> filterRelevant
    else haystack
```

### `case-of`

- Add a newline after the `case-of` expression and each case's `->`

```elm
-- ✅ GOOD
case thing of
    Diamond diamond ->
        wow diamond

    PocketLint ->
        oh


-- ❌ BAD
case thing of
  Diamond diamond  -> wow diamond
  PocketLint       -> oh
```

The `case thing of` clause should be one line down for glanceability.

"Rhythmic" indentations for the `->` arrows may look nice, but they quickly become a nightmare for maintainability. Simply adding a parameter to any of the cases can push the arrow on that line further than the others, requiring modifications on all lines.

As of 0.16 elm-compiler can check for non-exhaustive case matches!


## Module definitions

- Import only needed modules
- Order of preference:
  1. non-exposing imports
  2. explicitly exposing imports
  3. imports exposing everything
- When feasible, explicitly define what to expose from current module

```elm
import Best -- ✅ GOOD
import Okay exposing (This, That)
import NotGood exposing (..)
```

## Declarations

- Name functions and parameters descriptively
- Write type annotations
- Prefer `|>` (each on new line), starting with whatever feels most natural
- Avoid long functions
- Split long `let` blocks into separate functions
- Always add a newline after an equals sign `=`

```elm
-- ✅ GOOD
maybeToList : Maybe a -> List a
maybeToList maybe =
    maybe
        |> Maybe.map (List.repeat 1)
        |> Maybe.withDefault []

-- ❌ BAD
import ?? exposing (..)
mtl m = withDefault [] (map (repeat 1) m)
```

- Using imports with `exposing (..)` pollutes the namespace and makes it impossible to know which module is responsible for the functions `map` and `repeat` for example.
- Missing type annotation makes matters even worse.
- Indescriptive names cause mental overhead.
- No newline after equals sign will lead to 1) long lines and 2) worse version control diffs.
- Parens syntax is harder to glance through than `|>`.


## Working with `elm-html`

`elm-html` is the exception to the `import ?? exposing (..)` rule. It is very nice use the functions without the fully qualified names. Still, even with the Html modules, it's totally feasible to import only what you really need.

We found it a good practice to start building the layout for a UI component with a simple skeleton such as the following:

```elm
module Sample (view) where

import Html exposing (Html, div, text)
import Html.Attributes exposing (class)

import Model exposing (Model)

view : Model -> Html msg
view model =
    div
        [ class "sample"
        ]
        [ text "Sample View"
        ]
```

It is then easy to add helper functions that won't be exposed to other modules.


### Dealing with the lists

All elements in `elm-html` are functions with two lists as parameters. This means you'll be dealing with lists a lot. Building up components from several helper functions can quickly become unwieldy, unless you have decided upon standard ways of doing it.

The two main methods we found best are 1) concatenating the child list, and 2) mapping a list.

Concatenating a list with `++` gives great flexibility: based on certain conditions (like logged-in state), the children of a top-level component might be totally - or partly - different.

When rendering a list of things, however, it makes a ton of sense to just map over the list to render it.

```elm
-- Different kinds of children: concat
consChildren : Model -> Html
consChildren model =
    div
        [ --attributes
        ]
        <| [ Header.view model.title ]
        ++ viewMaybe model.subTitle
        ++ [ Footer.view model.footerThings ]


-- Same kinds of children: map
mapChildren : List Thing -> Html
mapChildren things =
    div
        [ --attributes
        ]
        <| List.map viewThing things
```

Now, you might think the `consChildren` example is weird with the single-item lists and the `viewMaybe` thing in the middle. I admit, it is a bit strange at first. It would be nice to write it just like:

```elm
[ Header.view model.title
, viewSubTitle model.subTitle
, Footer.view model.footerThings
]
```

But as said, the concat structure brings great flexibility. The `Header.view` and `Footer.view` functions return just plain `Html`, but the `viewMaybe` function returns a list. This is very convenient, because a list can always be empty and still work. Assuming `model.subTitle` is a Maybe type, `viewSubTitle` would have to always return some phony `Html` just to match the type, even when it shouldn't render at all.


## License

This style guide is © Ossi Hanhinen and licensed under the [MIT License](LICENSE).
