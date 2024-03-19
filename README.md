# osakit

`osakit` aims to provide direct access to `OSAKit Framework` of macOS. Is uses ObjC-bindings
to access OSAKit and run both `AppleScript` and `JavaScript`.

`osakit` is built using [`serde`](https://crates.io/crates/serde) for input-output
serialization/deserialization.
Allows passing data to `JavaScript`/`AppleScript` functions and returns back the results.
Input and output data are represented using `Value` from
[`serde_json`](https://crates.io/crates/serde_json).

Comes with `declare_script!` macro (unstable) to simplify working with `OSAKit Framework`.

## Installation

Add `osakit` to the dependencies. Specify `"full"` feature if you want to use `declare_script`
macro or `"stable"` feature to only include stable API.

```toml
[dependencies]
osakit = { version = "0.1.0", features = ["full"] }
```

## Example using `declare_script`

```rust
use serde::{Deserialize, Serialize};
use osakit::declare_script;
                                                                                                       
declare_script! {
    #[language(JavaScript)]
    #[source("
        function concat(x, y) {
            return x + y;
        }
                                                                                                       
        function multiply(a, b) {
            return a * b;
        }
                                                                                                       
        function current_user() {
            return {
                id: 21,
                name: \"root\"
            };
        }
    ")]
    MyJsScript {
        fn concat(x: &str, y: &str) -> String;
        fn multiply(a: i32, b: i32) -> i32;
        fn current_user() -> User;
    }
}
                                                                                                       
#[derive(Deserialize, Serialize, Debug, PartialEq, Eq)]
struct User {
    id: u16,
    name: String,
}
                                                                                                       
#[test]
fn it_runs_my_js_script() {
    let script = MyJsScript::new().unwrap();
    assert_eq!(
        script.multiply(3, 2).unwrap(),
        6
    );
    assert_eq!(
        script.concat("Hello, ", "World").unwrap(),
        "Hello, World"
    );
    assert_eq!(
        script.current_user().unwrap(),
        User {
            id: 21,
            name: "root".into()
        }
    );
}
```

## Example using `Script`

```rust
use osakit::{Language, Map, Script, Value, Number};
                                                                                                       
#[test]
fn it_constructs_and_executes_scripts() {
    let mut script = Script::new_from_source(
        Language::AppleScript, "
        on launch_terminal()
            tell application \"Terminal\" to launch
        end launch_terminal
                                                                                                       
        on concat(x, y)
            return x & y
        end concat
                                                                                                       
        return {id: 21, name: \"root\"}
    ");
                                                                                                       
    script.compile().unwrap();
                                                                                                       
    assert_eq!(
        script.execute().unwrap(),
        Value::Object(Map::from_iter(vec![
            ("id".into(), Value::Number(Number::from(21))),
            ("name".into(), Value::String("root".into()))
        ]))
    );
                                                                                                       
    assert_eq!(
        script.execute_function("concat", &vec![
            Value::String("Hello, ".into()),
            Value::String("World!".into())
        ]).unwrap(),
        Value::String("Hello, World!".into())
    );
                                                                                                       
    assert!(
        script.execute_function("launch_terminal", &vec![]).is_ok()
    );
}
```

## Usage

See [Full Documentation](https://docs.rs/osakit/).

## Supported platforms

Due to the fact that OSAKit is Mac-specific, only `macOS` is supported.

## License

MIT
Copyright (c) 2024 Marat Dulin
