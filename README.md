# potoo-potigol-interpreter

A Rust interpreter library for the documented Potigol 1.0.0-RC2 language core. It depends on `fancy-regex` and can be embedded in Rust applications; it includes a standalone terminal command-line runner. It is an independent implementation and does not run or bundle the official Scala/JVM interpreter.

## Source example

```poti
escreva "Olá, Potigol!"
```

## Use

Clone this repository and add the crate to a Rust application with a Git dependency pinned to the `v0.1.0` release (in the application's `Cargo.toml`):

```toml
[dependencies]
potoo-potigol-interpreter = { git = "https://github.com/diegopn/potoo-potigol-interpreter.git", tag = "v0.1.0" }
```

Implement `RuntimeIo` to provide input and output, then call `execute`:

```rust
use potoo_potigol_interpreter::{InputPrompt, RuntimeIo, execute};

struct Console;

impl RuntimeIo for Console {
    fn write(&mut self, text: &str) {
        print!("{text}");
    }

    fn read_line(&mut self, _prompt: &InputPrompt) -> Result<String, String> {
        let mut input = String::new();
        std::io::stdin()
            .read_line(&mut input)
            .map_err(|error| error.to_string())?;
        Ok(input.trim_end_matches(['\r', '\n']).to_owned())
    }
}

fn main() -> Result<(), Vec<potoo_potigol_interpreter::Diagnostic>> {
    execute(
        "nome = leia_texto\nescreva \"Olá, {nome}!\"",
        &mut Console,
    )
}
```

Call `parse` when a host needs to inspect the public syntax tree without executing the program. `check` exposes static type checking. `execute_with_options` accepts per-run resource limits.

## Supported language features

- Type inference, typed declarations and parameters, primitive values, lists, mutable lists, tuples, and user-defined types
- Functions, closures, recursion, lambdas, currying, and higher-order list operations
- Pattern matching, constructor patterns, guards, conditionals, loops, and list comprehensions
- Typed input, formatted and interpolated output, numeric and text operations, and built-in math functions
- Abstract types, multiple inheritance, source-located diagnostics, cancellation, and debugger events for host applications

The compatibility target is the documented Potigol 1.0.0-RC2 language core. With `use "jerimum"`, the crate also provides a native compatibility layer for `Jogo`, `Teclado`, `Mouse`, `Relógio`, `Arquivo`, `Cor`, `Imagem`, `Animacao`, `Fonte`, `Som`, `Musica`, `Tela`, and `Desenho`.

Jerimum graphics, audio, input, window state, and file reads go through `RuntimeIo::jerimum_request`. The default implementation reports that the host does not support Jerimum; embedders must provide those services. Potoo's IDE maps the requests to its GTK graphics and audio hosts and confines file and media paths to the active project. Native facades support `Color` channel getters; `BufferedImage` dimensions, pixel reads, and subimages; `JFrame` title/dimensions; `Canvas` dimensions; and `Graphics2D` color, line, rectangle, oval, text, and image drawing. These are not JVM objects or general Swing/AWT interop; unsupported Java methods remain unavailable.

## Runtime limits

By default, one execution is limited to 1,000,000 steps, 1 MiB of output, 1 MiB per string, 100,000 collection elements, an estimated cumulative 64 MiB of collection payload, 30 seconds of waiting, 128 nested calls, and 128 levels of parser nesting. Hosts can change these limits with `ExecutionOptions`.

The collection byte limit estimates cumulative payload allocation; it is not a strict live-heap ceiling. This interpreter runs in the host process and is not an operating-system sandbox. Hosts running untrusted source should apply appropriate process isolation.

## Development

Run these commands from the repository root:

```sh
cargo fmt --all -- --check
cargo check
cargo test
```

The conformance tests cover representative examples from Potigol documentation and release notes.

## Project policies

- [Contributing](CONTRIBUTING.md)
- [Security policy](SECURITY.md)
- [GPL-3.0-or-later license](LICENSE)

## Standalone command-line runner

The `potoo-potigol` command runs one source file without Potoo IDE. For example:

```sh
potoo-potigol example.poti
```

The terminal runner provides standard input/output. Jerimum graphics, audio, window, keyboard, mouse, and file callbacks are unavailable; the interpreter reports those requests as unsupported. The runner does not provide an interactive debugger.

## Install a prebuilt runner

Download the archive for your system and `SHA256SUMS` from the [latest GitHub Release](https://github.com/diegopn/potoo-potigol-interpreter/releases/latest). The first release is tagged `v0.1.0` and uses these asset names:

| System | Archive |
| --- | --- |
| Linux x86_64 | `potoo-potigol-interpreter-v0.1.0-x86_64-unknown-linux-gnu.tar.gz` |
| Windows x86_64 | `potoo-potigol-interpreter-v0.1.0-x86_64-pc-windows-msvc.zip` |
| macOS Intel | `potoo-potigol-interpreter-v0.1.0-x86_64-apple-darwin.tar.gz` |
| macOS Apple silicon | `potoo-potigol-interpreter-v0.1.0-aarch64-apple-darwin.tar.gz` |

### Linux x86_64

```sh
asset=potoo-potigol-interpreter-v0.1.0-x86_64-unknown-linux-gnu.tar.gz
release=https://github.com/diegopn/potoo-potigol-interpreter/releases/download/v0.1.0
curl -fLO "$release/$asset"
curl -fLO "$release/SHA256SUMS"
sha256sum "$asset" # compare with its line in SHA256SUMS
tar -xzf "$asset"
mkdir -p "$HOME/.local/bin"
install -m 755 "potoo-potigol-interpreter-v0.1.0-x86_64-unknown-linux-gnu/potoo-potigol" "$HOME/.local/bin/potoo-potigol"
potoo-potigol example.poti
```

Add it for the current shell with `export PATH="$HOME/.local/bin:$PATH"`; place that line in your shell profile to keep it for future sessions.

### Windows x86_64 (PowerShell)

```powershell
$asset = "potoo-potigol-interpreter-v0.1.0-x86_64-pc-windows-msvc.zip"
$release = "https://github.com/diegopn/potoo-potigol-interpreter/releases/download/v0.1.0"
Invoke-WebRequest "$release/$asset" -OutFile $asset
Invoke-WebRequest "$release/SHA256SUMS" -OutFile SHA256SUMS
Get-FileHash ".\$asset" -Algorithm SHA256 # compare with the matching SHA256SUMS entry
Expand-Archive ".\$asset" -DestinationPath .\potoo-potigol-interpreter
.\potoo-potigol-interpreter\potoo-potigol-interpreter-v0.1.0-x86_64-pc-windows-msvc\potoo-potigol.exe .\example.poti
```

To run `potoo-potigol.exe` from any folder, add its extracted directory to your user `PATH` in Windows Environment Variables.

### macOS Intel and Apple silicon

Use the `x86_64-apple-darwin` archive on Intel Macs and `aarch64-apple-darwin` on Apple silicon.

```sh
asset=potoo-potigol-interpreter-v0.1.0-<target>.tar.gz
release=https://github.com/diegopn/potoo-potigol-interpreter/releases/download/v0.1.0
curl -fLO "$release/$asset"
curl -fLO "$release/SHA256SUMS"
shasum -a 256 "$asset" # compare with its line in SHA256SUMS
tar -xzf "$asset"
mkdir -p "$HOME/.local/bin"
install -m 755 "${asset%.tar.gz}/potoo-potigol" "$HOME/.local/bin/potoo-potigol"
potoo-potigol example.poti
```

Replace `<target>` with `x86_64-apple-darwin` or `aarch64-apple-darwin`. Add it for the current shell with `export PATH="$HOME/.local/bin:$PATH"`; place that line in `~/.zprofile` to keep it for future macOS sessions.

## Install from source

With Rust 1.93 or later and network access, Cargo can install the command directly from its tagged source:

```sh
cargo install --git https://github.com/diegopn/potoo-potigol-interpreter.git --tag v0.1.0 --bin potoo-potigol
```

## Use as a Rust library

```toml
[dependencies]
potoo-potigol-interpreter = { git = "https://github.com/diegopn/potoo-potigol-interpreter.git", tag = "v0.1.0" }
```

The library API lets Rust applications provide their own `RuntimeIo` implementation.

## DOI

A stable Figshare DOI will be added when the first public release is published.
