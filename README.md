# aegean

[![crates.io](https://img.shields.io/crates/v/aegean.svg)](https://crates.io/crates/aegean)
[![crates.io](https://docs.rs/aegean/badge.svg)](https://docs.rs/aegean)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue.svg)](https://github.com/gabrielbrunop/aegean)
![actions-badge](https://github.com/gabrielbrunop/aegean/workflows/Rust/badge.svg?branch=main)

A fancy compiler diagnostics crate.

## About

`aegean` is a fork of [ariadne](https://github.com/zesterer/ariadne) with additional features and improvements.

## Example

<a href = "https://github.com/gabrielbrunop/aegean/blob/main/examples/multiline.rs">
<img src="https://raw.githubusercontent.com/gabrielbrunop/aegean/main/misc/example.png" alt="aegean supports arbitrary multi-line spans"/>
</a>

```rust,ignore
fn main() {
    use aegean::{Color, ColorGenerator, Fmt, Label, Report, ReportKind, Source};

    let mut colors = ColorGenerator::new();

    // Generate & choose some colours for each of our elements
    let a = colors.next();
    let b = colors.next();
    let out = Color::Fixed(81);

    Report::build(ReportKind::Error, ("sample.tao", 12..12))
        .with_code(3)
        .with_message(format!("Incompatible types"))
        .with_label(
            Label::new(("sample.tao", 32..33))
                .with_message(format!("This is of type {}", "Nat".fg(a)))
                .with_color(a),
        )
        .with_label(
            Label::new(("sample.tao", 42..45))
                .with_message(format!("This is of type {}", "Str".fg(b)))
                .with_color(b),
        )
        .with_label(
            Label::new(("sample.tao", 11..48))
                .with_message(format!(
                    "The values are outputs of this {} expression",
                    "match".fg(out),
                ))
                .with_color(out),
        )
        .with_note(format!(
            "Outputs of {} expressions must coerce to the same type",
            "match".fg(out)
        ))
        .finish()
        .print(("sample.tao", Source::from(include_str!("sample.tao"))))
        .unwrap();
}
```

See [`examples/`](https://github.com/gabrielbrunop/aegean/tree/main/examples) for more examples.

## Usage

For each error you wish to report:

- Call `Report::build()` to create a `ReportBuilder`.
- Assign whatever details are appropriate to the error using the various
  methods, and then call the `finish` method to get a `Report`.
- For each `Report`, call `print` or `eprint` to write the report
  directly to `stdout` or `stderr`. Alternately, you can use
  `write` to send the report to any other `Write` destination (such as a file).

## Features

- Inline and multi-line labels capable of handling arbitrary configurations of spans
- Multi-file errors
- Generic across custom spans and file caches
- A choice of character sets to ensure compatibility
- Coloured labels & highlighting with 8-bit and 24-bit color support (thanks to
  [`yansi`](https://github.com/SergioBenitez/yansi))
- Label priority and ordering
- Compact mode for smaller diagnostics
- Correct handling of variable-width characters such as tabs
- A `ColorGenerator` type that generates distinct colours for visual elements.
- A plethora of other options (tab width, label attach points, underlines, etc.)
- Built-in ordering/overlap heuristics that come up with the best way to avoid overlapping & label crossover

## Cargo Features

- `"concolor"` enables integration with the [`concolor`](https://crates.io/crates/concolor) crate for global color output
  control across your application
- `"auto-color"` enables `concolor`'s `"auto"` feature for automatic color control

`concolor`'s features should be defined by the top-level binary crate, but without any features enabled `concolor` does
nothing. If `aegean` is your only dependency using `concolor` then `"auto-color"` provides a convenience to enable
`concolor`'s automatic color support detection, i.e. this:

```TOML
[dependencies]
aegean = { version = "...", features = ["auto-color"] }
```

is equivalent to this:

```TOML
[dependencies]
aegean = { version = "...", features = ["concolor"] }
concolor = { version = "...", features = ["auto"] }
```

## Stability

The API (should) follow [semver](https://www.semver.org/). However, this does not apply to the layout of final error
messages. Minor tweaks to the internal layout heuristics can often result in the exact format of error messages changing with labels moving slightly. If you experience a change in layout that you believe to be a regression (either the change is incorrect, or makes your diagnostics harder to read) then please open an issue.
