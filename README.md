# teashop

[![Package Version](https://img.shields.io/hexpm/v/teashop)](https://hex.pm/packages/teashop)
[![Hex Docs](https://img.shields.io/badge/hex-docs-ffaff3)](https://hexdocs.pm/teashop/)

## Tutorial

### Getting Started

For this tutorial, let's build a tea shop.

We'll start with creating a new Gleam project:

```
gleam new shop_tutorial
```

<!-- Update this when git dependencies or hex release -->
Then we add `teashop` to our project by adding it as a path dependency in our `gleam.toml`:

```toml
[dependencies]
teashop = { path = "../teashop" }
```

Now we can open up `src/shop_tutorial.gleam` and import the modules we'll need:

```gleam
// Teashop modules
import teashop
import teashop/event
import teashop/command
import teashop/key

// Gleam standard library modules
import gleam/list
import gleam/string
```

Teashop programs are composed of 3 parts:

- `init`, that specifies an initial model and any commands to be run right after startup
- `update`, that handles incoming events and updates the model accordingly
- `view`, that turns your model into a string to be rendered 

### The Model

```gleam
pub type Status {
  Selected
  Unselected
}

pub type Model {
  Model(choices: List(#(String, Status)), cursor: Int)
}
```

### Initialization

```gleam
const initial_model = Model(
  cursor: 0,
  choices: [
    #("Kitten cuddles 🐈", Unselected),
    #("Strawberry shortcake 🍰", Unselected),
    #("Blueberry muffins 🫐", Unselected),
  ],
)

pub fn init(_) {
  #(initial_model, command.set_window_title("teashop"))
}
```

### Update

```gleam
pub fn update(model: Model, event) {
  case event {
    event.Key(key.Char("q")) | event.Key(key.Esc) -> #(model, command.quit())

    event.Key(key.Char("k")) | event.Key(key.Up) -> {
      let choices_len = list.length(model.choices)
      let cursor = case model.cursor == 0 {
        True -> choices_len - 1
        False -> model.cursor - 1
      }
      #(Model(..model, cursor: cursor), command.noop())
    }

    event.Key(key.Char("j")) | event.Key(key.Down) -> {
      let choices_len = list.length(model.choices)
      let cursor = case model.cursor == { choices_len - 1 } {
        True -> 0
        False -> model.cursor + 1
      }
      #(Model(..model, cursor: cursor), command.noop())
    }

    event.Key(key.Enter) | event.Key(key.Space) -> {
      let toggle = fn(status) {
        case status {
          Selected -> Unselected
          Unselected -> Selected
        }
      }
      let choices =
        list.index_map(model.choices, fn(element, index) {
          let #(name, status) = element
          let status = case index == model.cursor {
            True -> toggle(status)
            False -> status
          }
          #(name, status)
        })
      #(Model(..model, choices: choices), command.noop())
    }
    _otherwise -> #(model, command.noop())
  }
}
```

### View

```gleam
pub fn view(model: Model) {
  let options =
    model.choices
    |> list.index_map(fn(element, index) {
      let #(name, status) = element
      let cursor = case model.cursor == index {
        True -> ">"
        False -> " "
      }
      let checked = case status {
        Selected -> "x"
        _ -> " "
      }
      cursor <> " [" <> checked <> "] " <> name
    })
    |> string.join("\n")

  let header = "What should we get at the tea shop?"
  let footer = "Press q to quit."

  [header, options, footer]
  |> string.join("\n\n")
}
```

### All Together Now

```gleam
pub fn main() {
  let app = teashop.app(init, update, view)
  teashop.start(app, Nil)
}
```

### Running our teashop:

```
gleam run -m shop_tutorial --target js
```
