# minimal-rust-std

**Reduce your hello-world build size by 10x**

> Problem: A normal hello-world build in rust comes in at around 300kB+ (on
> Windows at least), being pretty heavy for this simple task or tiny command
> line utilities. A lot of that weight comes from stack unwinding logic and
> panic handling.

This setup template reduces the hello-world binary size to 20kB without tapping
into the unwieldy territory of `no_std` or `no_main` letting you write Rust as
you used to without needing to resort to assembly or typing out your own
syscall-signatures.

Keep in mind this uses aggressive release profile settings (`strip`, `lto`,
`opt-level "z"`, `panic = "immediate-abort"`) to produce a minimal executable.
In case your executable panics it will simply fail with an exit code of 1.
That's it.

## Usage

After cloning this repo run

```bash
cargo build --release
```

or

```bash
cargo run --release
```

The output binary will be in `target/release/`.

> Note: Upon first running those `cargo` might have to download the latest
> nightly Rust release which might take time if you do not have it installed
> already.

## Configuration Explainer

This setup uses three configuration files, each essential. They work together to
recompile the standard library from source with aggressive size optimizations.

### `rust-toolchain.toml`

Controls which Rust toolchain and components Cargo uses for this project.

| Setting      | Value          | Purpose                                                                                                  |
| ------------ | -------------- | -------------------------------------------------------------------------------------------------------- |
| `channel`    | `"nightly"`    | Uses the nightly compiler, required for unstable features like `build-std` and `panic-immediate-abort`   |
| `components` | `["rust-src"]` | Installs the standard library source code locally, which is needed to recompile `std`/`core` from source |

### `.cargo/config.toml`

Configures Cargo's build behaviour for this project, enabling standard library
rebuilds.

| Setting                 | Value                            | Purpose                                                                                                                                                                                                                                                                               |
| ----------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `build-std`             | `["std", "core", "panic_abort"]` | Rebuilds the standard library (`std`), core language primitives (`core`), and the panic runtime (`panic_abort`) from source instead of using the precompiled versions. This allows the size optimizations in `Cargo.toml` to apply to the standard library itself, not just your code |
| `panic-immediate-abort` | `true`                           | Instructs the compiler to replace all panic paths with an immediate `abort()` call, removing the formatting and unwinding machinery that normally ships with panics. This is the single largest contributor to the size reduction                                                     |

### `Cargo.toml`

Defines the project metadata, dependencies, and the release profile that reduces
the final binary size down to the mentioned 20kB.

| Setting          | Value                       | Purpose                                                                                                                                         |
| ---------------- | --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `cargo-features` | `["panic-immediate-abort"]` | Enables the unstable `panic-immediate-abort` Cargo feature, which is required for the `panic = "immediate-abort"` profile setting below         |
| `strip`          | `true`                      | Removes debug symbols and symbol tables from the binary                                                                                         |
| `opt-level`      | `"z"`                       | Tells LLVM to aggressively optimize for smallest binary size (more aggressive than the default `"s"`)                                           |
| `codegen-units`  | `1`                         | Compiles the entire crate in a single codegen unit, giving LLVM full visibility for optimization at the cost of slower builds                   |
| `lto`            | `true`                      | Enables link-time optimization, allowing the linker to optimize across all compiled units together                                              |
| `panic`          | `"immediate-abort"`         | Combined with the `panic-immediate-abort` unstable feature, this eliminates all panic formatting and stack unwinding code from the final binary |

## Further Reads

For more details into bin size optimizations refer to this excellent repo:

[min-sized-rust by johnthagen](https://github.com/johnthagen/min-sized-rust)

Also, if you want a really minimal build and don't mind writing unsafe rust or
your own syscall templates, check out he current minimalist build at ~260 bytes
here:

[min-sized-rust-windows by mcountryman](https://github.com/mcountryman/min-sized-rust-windows)

## Contributions Welcome

You found another tweak to minimize the build size that _does not_ introduce
`no_std`, `no_main`, `unsafe` or `extern "C"` in the source? Open an issue or
pull request and share the knowledge!

## License

[MIT](LICENSE)
