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

The crate depends on `fancy-regex` and can be embedded in Rust applications. It does not run or bundle the official Scala/JVM interpreter.

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

The `potoo-potigol` command runs one source file without Potoo IDE. For example:

```sh
potoo-potigol example.poti
```

The terminal runner provides standard input/output. Jerimum graphics, audio, window, keyboard, mouse, and file callbacks are unavailable; the interpreter reports those requests as unsupported. The runner does not provide an interactive debugger.

## Install a prebuilt runner

Prebuilt binaries are published on the [GitHub Releases page](https://github.com/diegopn/potoo-potigol-interpreter/releases). Choose a version and the package for your system there. Packages are currently available for Linux x86_64, Windows x86_64 (MSVC), and Intel macOS (x86_64). The current release does not include an Apple silicon (arm64) package.

Download only the package for your system and the `SHA256SUMS` file, and save both in the same folder. Before extracting or running the executable, compare the package's SHA-256 hash with its corresponding entry in `SHA256SUMS`. Run the commands below from that folder. Replace `example.poti` with the path to a source file you want to run; the release package does not include a sample file.

### Linux x86_64

```sh
asset=$(ls -1 potoo-potigol-interpreter-v*-x86_64-unknown-linux-gnu.tar.gz | head -n 1)
sha256sum "$asset" # compare with its line in SHA256SUMS
tar -xzf "$asset"
mkdir -p "$HOME/.local/bin"
install -m 755 "${asset%.tar.gz}/potoo-potigol" "$HOME/.local/bin/potoo-potigol"
export PATH="$HOME/.local/bin:$PATH"
potoo-potigol example.poti
```

To keep the command available in new Linux sessions, add the `export PATH="$HOME/.local/bin:$PATH"` line to your shell profile.

### Windows x86_64 (PowerShell)

```powershell
$archive = Get-ChildItem -Filter "potoo-potigol-interpreter-v*-x86_64-pc-windows-msvc.zip" | Select-Object -First 1
Get-FileHash $archive.FullName -Algorithm SHA256 # compare with the matching SHA256SUMS entry
Expand-Archive $archive.FullName -DestinationPath .\potoo-potigol
$exe = Get-ChildItem .\potoo-potigol -Filter "potoo-potigol.exe" -Recurse | Select-Object -First 1
& $exe.FullName .\example.poti
```

To run `potoo-potigol.exe` from another folder, add the directory containing the executable to your user `PATH`.

### Intel macOS

The published macOS package targets Intel Macs (x86_64). There is no prebuilt Apple silicon package in the current release.

```sh
asset=$(ls -1 potoo-potigol-interpreter-v*-x86_64-apple-darwin.tar.gz | head -n 1)
shasum -a 256 "$asset" # compare with its line in SHA256SUMS
tar -xzf "$asset"
mkdir -p "$HOME/.local/bin"
install -m 755 "${asset%.tar.gz}/potoo-potigol" "$HOME/.local/bin/potoo-potigol"
export PATH="$HOME/.local/bin:$PATH"
potoo-potigol example.poti
```

Add the `export PATH="$HOME/.local/bin:$PATH"` line to `~/.zprofile` to keep the command available in future macOS sessions.


## Install from source

With Rust 1.93 or later and network access, Cargo can install the command directly from its tagged source:

```sh
cargo install --git https://github.com/diegopn/potoo-potigol-interpreter.git --rev ab2c74af7cec16f5dd2a5bff2391a01f0b61001f --bin potoo-potigol
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
