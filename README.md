Anyhow&ensp;¯\\\_(ツ)\_/¯
=========================

[![Build Status](https://api.travis-ci.com/dtolnay/eyre.svg?branch=master)](https://travis-ci.com/dtolnay/eyre)
[![Latest Version](https://img.shields.io/crates/v/eyre.svg)](https://crates.io/crates/eyre)
[![Rust Documentation](https://img.shields.io/badge/api-rustdoc-blue.svg)](https://docs.rs/eyre)

This library provides [`eyre::Error`][Error], a trait object based error type
for easy idiomatic error handling in Rust applications.

[Error]: https://docs.rs/eyre/1.0/eyre/struct.Error.html

```toml
[dependencies]
eyre = "1.0"
```

*Compiler support: requires rustc 1.34+*

<br>

## Details

- Use `Result<T, eyre::Error>`, or equivalently `eyre::Result<T>`, as the
  return type of any fallible function.

  Within the function, use `?` to easily propagate any error that implements the
  `std::error::Error` trait.

  ```rust
  use eyre::Result;

  fn get_cluster_info() -> Result<ClusterMap> {
      let config = std::fs::read_to_string("cluster.json")?;
      let map: ClusterMap = serde_json::from_str(&config)?;
      Ok(map)
  }
  ```

- Attach context to help the person troubleshooting the error understand where
  things went wrong. A low-level error like "No such file or directory" can be
  annoying to debug without more context about what higher level step the
  application was in the middle of.

  ```rust
  use eyre::{Report, Result};

  fn main() -> Result<()> {
      ...
      it.detach().context("Failed to detach the important thing")?;

      let content = std::fs::read(path)
          .with_context(|| format!("Failed to read instrs from {}", path))?;
      ...
  }
  ```

  ```console
  Error: Failed to read instrs from ./path/to/instrs.json

  Caused by:
      No such file or directory (os error 2)
  ```

- Downcasting is supported and can be by value, by shared reference, or by
  mutable reference as needed.

  ```rust
  // If the error was caused by redaction, then return a
  // tombstone instead of the content.
  match root_cause.downcast_ref::<DataStoreError>() {
      Some(DataStoreError::Censored(_)) => Ok(Poll::Ready(REDACTED_CONTENT)),
      None => Err(error),
  }
  ```

- A backtrace is captured and printed with the error if the underlying error
  type does not already provide its own. In order to see backtraces, the
  `RUST_LIB_BACKTRACE=1` environment variable must be defined.

- Anyhow works with any error type that has an impl of `std::error::Error`,
  including ones defined in your crate. We do not bundle a `derive(Error)` macro
  but you can write the impls yourself or use a standalone macro like
  [thiserror].

  ```rust
  use thiserror::Error;

  #[derive(Error, Debug)]
  pub enum FormatError {
      #[error("Invalid header (expected {expected:?}, got {found:?})")]
      InvalidHeader {
          expected: String,
          found: String,
      },
      #[error("Missing attribute: {0}")]
      MissingAttribute(String),
  }
  ```

- One-off error messages can be constructed using the `eyre!` macro, which
  supports string interpolation and produces an `eyre::Error`.

  ```rust
  return Err(eyre!("Missing attribute: {}", missing));
  ```

<br>

## No-std support

In no_std mode, the same API is almost all available and works the same way. To
depend on Anyhow in no_std mode, disable our default enabled "std" feature in
Cargo.toml. A global allocator is required.

```toml
[dependencies]
eyre = { version = "1.0", default-features = false }
```

Since the `?`-based error conversions would normally rely on the
`std::error::Error` trait which is only available through std, no_std mode will
require an explicit `.map_err(Error::msg)` when working with a non-Anyhow error
type inside a function that returns Anyhow's error type.

<br>

## Comparison to failure

The `eyre::Error` type works something like `failure::Error`, but unlike
failure ours is built around the standard library's `std::error::Error` trait
rather than a separate trait `failure::Fail`. The standard library has adopted
the necessary improvements for this to be possible as part of [RFC 2504].

[RFC 2504]: https://github.com/rust-lang/rfcs/blob/master/text/2504-fix-error.md

<br>

## Comparison to thiserror

Use Anyhow if you don't care what error type your functions return, you just
want it to be easy. This is common in application code. Use [thiserror] if you
are a library that wants to design your own dedicated error type(s) so that on
failures the caller gets exactly the information that you choose.

[thiserror]: https://github.com/dtolnay/thiserror

<br>

#### License

<sup>
Licensed under either of <a href="LICENSE-APACHE">Apache License, Version
2.0</a> or <a href="LICENSE-MIT">MIT license</a> at your option.
</sup>

<br>

<sub>
Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this crate by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
</sub>
