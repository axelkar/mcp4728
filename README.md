# Rust driver for MCP4728 4-channel 12-bit I2C DAC

[![Test][test-badge]][test-link]
[![Crates.io][crates-badge]][crates-link]
[![Docs.rs][docs-badge]][docs-link]

This is a platform agnostic rust driver for the MCP4728 DAC using the
[embedded-hal](https://github.com/rust-embedded/embedded-hal) and optionally
the [embedded-hal-async](https://docs.rs/embedded-hal-async/latest/embedded_hal_async/) traits.

This driver allows you to:

- Write to a single channel and save to EEPROM
- Write to multiple channels and save to EEPROM
- Write to all channels, skipping some configuration bits and not saving to EEPROM ("fast write")
- Write voltage reference mode to all channels
- Write gain mode to all channels
- Write power down mode to all channels
- Read EEPROM and output registers for all channels
- Issue I2C General Call commands: reset, wake up, and software update

There is one feature that this driver does **not** support - reading and writing the I2C address of
the device. Three bits of the driver can be set (from 0x60 to 0x67) but this is done by toggling
another pin in precise timing with the I2C message, which is not possible to do using the
[embedded-hal](https://github.com/rust-embedded/embedded-hal) traits.

This crate is `#![no_std]` and does not contain unsafe code.

## Device

The MCP4728 is a 4-channel, 12-bit digital-to-analog converter with nonvolatile memory (EEPROM) for
its output settings. The device will load its previous setting from EEPROM on startup without
receiving any commands. Each channel has separate voltage reference, gain, and power down settings.
Channels can be written individually or together.

### Datasheet

[MCP4728](https://ww1.microchip.com/downloads/en/DeviceDoc/22187E.pdf)

## Optional Cargo features

* `defmt-03`: Derive [`defmt::Format`](https://docs.rs/defmt/0.3.8/defmt/trait.Format.html) from `defmt` 0.3 for some enums and structs
* `async`: Use the [`embedded-hal-async`](https://docs.rs/embedded-hal-async/latest/embedded_hal_async/) traits for the `MCP4728` struct

## Usage

```rust
use linux_embedded_hal::I2cdev;
use mcp4728::{MCP4728};

let i2c = I2cdev::new("/dev/i2c-1").unwrap();
let mut dac = MCP4728::new(i2c, 0x60);
dac.fast_write(483, 279, 297, 590).unwrap();
```

### Updating the analog outputs

Most commands write to the output register but do not necessarily update the analog output.
There are several ways to do so:

- If `OutputEnableMode` is `Update`, the output will be updated after the last byte is received for
  each channel (not all commands can set this bit)
- If the LDAC pin is set low, the output will be updated after the last byte is received for each
  channel.
- If the LDAC pin transitions from high to low at any time, all channels will be updated.
- If a General Call Software Update command is received, all channels will be updated.

## Minimum Supported Rust Version (MSRV)

This crate is guaranteed to compile on stable Rust 1.65 and up. It _might_ compile with older
versions but that may change in any new patch release.

With the `async` feature on, `embedded-hal-async` has an MSRV of 1.75.

## License

Licensed under either of

- Apache License, Version 2.0
  ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license
  ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.

<!-- Badges -->

[test-badge]: https://github.com/davidstalnaker/mcp4728/actions/workflows/test.yaml/badge.svg
[test-link]: https://github.com/davidstalnaker/mcp4728/actions/workflows/test.yaml
[crates-badge]: https://img.shields.io/crates/v/mcp4728
[crates-link]: https://crates.io/crates/mcp4728
[docs-badge]: https://img.shields.io/docsrs/mcp4728
[docs-link]: https://docs.rs/mcp4728/latest/mcp4728
