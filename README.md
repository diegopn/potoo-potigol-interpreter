<p align="center">
  <img src="https://diegopn.github.io/potoo-site/src/assets/images/language-interpreter-icon.svg?v=1c651ca7" alt="Potoo IDE language interpreter icon" width="120" />
</p>

<h1 align="center">Potigol Interpreter</h1>

<p align="center">An independent Rust interpreter library and CLI for the documented Potigol 1.0.0-RC2 language core.</p>

<p align="center">
  <a href="https://github.com/diegopn/potoo-potigol-interpreter/releases/latest">Download</a> ·
  <a href="https://diegopn.github.io/potoo-site/">Website</a> ·
  <a href="https://diegopn.github.io/potoo-site/manual/potigol/">User manual</a>
</p>

<p align="center">
  <a href="https://www.rust-lang.org/"><img alt="Rust" src="https://img.shields.io/badge/Rust-000000?style=for-the-badge&logo=rust&logoColor=white"></a>
  <a href="https://doc.rust-lang.org/cargo/"><img alt="Cargo" src="https://img.shields.io/badge/Cargo-DEA584?style=for-the-badge&logo=rust&logoColor=white"></a>
  <img alt="Rust library" src="https://img.shields.io/badge/Rust_library-4A86CF?style=for-the-badge">
  <img alt="CLI" src="https://img.shields.io/badge/CLI-6A4C93?style=for-the-badge">
</p>

Originally developed as part of the Potoo IDE project, this interpreter is an independent Rust software package intended to be distributed separately from the IDE. It can be embedded as a library in other Rust projects or run as a standalone command-line tool on Linux, macOS, and Windows.

The crate depends on `fancy-regex` and can be embedded in Rust applications. It does not run or bundle the official Scala/JVM interpreter.

## Original language references

- [Potigol language documentation](https://potigol.github.io/docs/)
- [Upstream Potigol interpreter](https://github.com/potigol/potigol)
- [Documentation source repository](https://github.com/potigol/docs)

These are the original language and implementation references for compatibility work. This repository provides an independent Rust implementation.

## Original language references

- [Potigol language documentation](https://potigol.github.io/docs/)
- [Upstream Potigol interpreter](https://github.com/potigol/potigol)
- [Documentation source repository](https://github.com/potigol/docs)

These are the original language and implementation references for compatibility work. This repository provides an independent Rust implementation.

## Source example

```poti
escreva "Olá, Potigol!"
```

## Use

Clone this repository and add the crate to a Rust application with a Git dependency pinned to commit `ab2c74af7cec16f5dd2a5bff2391a01f0b61001f` (in the application's `Cargo.toml`):

```toml
[dependencies]
potoo-potigol-interpreter = { git = "https://github.com/diegopn/potoo-potigol-interpreter.git", rev = "ab2c74af7cec16f5dd2a5bff2391a01f0b61001f" }
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

## Support

If this interpreter is useful in your work, you can support its continued development:

<p align="left">
  <a href="https://github.com/sponsors/diegopn"><img alt="Become a sponsor" src="https://img.shields.io/badge/Become_a_sponsor-EA4AAA?style=for-the-badge&logo=githubsponsors&logoColor=white"></a>
</p>

## Project policies

- [Code of conduct](CODE_OF_CONDUCT.md)
- [Contributing](CONTRIBUTING.md)
- [Security policy](SECURITY.md)
- [GPL-3.0-or-later license](LICENSE)

## Standalone command-line runner

Save the Potigol source example above as `example.poti`, then run it with:

```sh
potoo-potigol example.poti
```

It prints `Olá, Potigol!`. The terminal runner provides standard input/output. Jerimum graphics, audio, window, keyboard, mouse, and file callbacks are unavailable; the interpreter reports those requests as unsupported. The runner does not provide an interactive debugger.

## Install a prebuilt runner

The v0.1.1 release provides native CLI packages for Linux x86_64 GNU, Windows x86_64 MSVC, Intel macOS x86_64, and Apple silicon macOS arm64. Download your system's archive and `SHA256SUMS` from the [latest GitHub release](https://github.com/diegopn/potoo-potigol-interpreter/releases/latest), then run the matching commands below from the folder containing both files. Each archive contains `potoo-potigol` (or `potoo-potigol.exe` on Windows) and the repository license. Use the `example.poti` source shown above or replace it with your own file.

### Linux x86_64 GNU

```sh
set -euo pipefail
archive="potoo-potigol-interpreter-v0.1.1-x86_64-unknown-linux-gnu.tar.gz"
grep -F "  $archive" SHA256SUMS | sha256sum --check -
tar -xzf "$archive"
mkdir -p "$HOME/.local/bin"
install -m 755 "${archive%.tar.gz}/potoo-potigol" "$HOME/.local/bin/potoo-potigol"
export PATH="$HOME/.local/bin:$PATH"
potoo-potigol example.poti
```

To keep the command available in new Linux sessions, add the `export PATH="$HOME/.local/bin:$PATH"` line to your shell profile.

### Windows x86_64 MSVC (PowerShell)

```powershell
$archive = "potoo-potigol-interpreter-v0.1.1-x86_64-pc-windows-msvc.zip"
$line = Get-Content .\SHA256SUMS | Where-Object { $_.EndsWith("  $archive") } | Select-Object -First 1
if (-not $line) { throw "No SHA256SUMS entry found for $archive" }
$expected = ($line -split '\s+')[0]
$actual = (Get-FileHash -LiteralPath $archive -Algorithm SHA256).Hash.ToLowerInvariant()
if ($actual -ne $expected) { throw "SHA-256 mismatch for $archive" }
$installDir = Join-Path $env:USERPROFILE ".local\bin"
New-Item -ItemType Directory -Force $installDir | Out-Null
Expand-Archive -LiteralPath $archive -DestinationPath $installDir
$exe = Join-Path $installDir "potoo-potigol-interpreter-v0.1.1-x86_64-pc-windows-msvc\potoo-potigol.exe"
& $exe .\example.poti
```

To run `potoo-potigol.exe` from any folder, add `$env:USERPROFILE\.local\bin\potoo-potigol-interpreter-v0.1.1-x86_64-pc-windows-msvc` to your user `Path` environment variable in Windows Settings.

### Intel macOS x86_64

```sh
set -euo pipefail
archive="potoo-potigol-interpreter-v0.1.1-x86_64-apple-darwin.tar.gz"
grep -F "  $archive" SHA256SUMS | shasum -a 256 -c -
tar -xzf "$archive"
mkdir -p "$HOME/.local/bin"
install -m 755 "${archive%.tar.gz}/potoo-potigol" "$HOME/.local/bin/potoo-potigol"
export PATH="$HOME/.local/bin:$PATH"
potoo-potigol example.poti
```

### Apple silicon macOS arm64

```sh
set -euo pipefail
archive="potoo-potigol-interpreter-v0.1.1-aarch64-apple-darwin.tar.gz"
grep -F "  $archive" SHA256SUMS | shasum -a 256 -c -
tar -xzf "$archive"
mkdir -p "$HOME/.local/bin"
install -m 755 "${archive%.tar.gz}/potoo-potigol" "$HOME/.local/bin/potoo-potigol"
export PATH="$HOME/.local/bin:$PATH"
potoo-potigol example.poti
```

On macOS, add the `export PATH="$HOME/.local/bin:$PATH"` line to `~/.zprofile` to keep the command available in future sessions.


## Install from source

With Rust 1.93 or later and network access, Cargo can install the command directly from its tagged source:

```sh
cargo install --git https://github.com/diegopn/potoo-potigol-interpreter.git --tag v0.1.1 --bin potoo-potigol
```

## Use as a Rust library

```toml
[dependencies]
potoo-potigol-interpreter = { git = "https://github.com/diegopn/potoo-potigol-interpreter.git", rev = "ab2c74af7cec16f5dd2a5bff2391a01f0b61001f" }
```

The library API lets Rust applications provide their own `RuntimeIo` implementation.

## DOI and citation

DOI: [10.6084/m9.figshare.34915038](https://doi.org/10.6084/m9.figshare.34915038)

Suggested reference (ABNT NBR 6023:2018):

> NASCIMENTO, Diego Pereira do. *Potigol interpreter v0.1.0*. 2026. Programa de computador. Disponível em: https://doi.org/10.6084/m9.figshare.34915038. Acesso em: 4 nov. 2026.
