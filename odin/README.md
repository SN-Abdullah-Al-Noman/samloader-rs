# samloader-odin

A safe and high-performance Rust implementation of the Samsung Odin/Loke flashing protocol driver and transport manager.

This crate manages communication with Samsung devices in Download Mode (Odin protocol), providing end-to-end support for handshake negotiation, partition mapping, and firmware uploads.

## Features

- Handles the full ODIN/Loke handshake, feature negotiation, and state management.
- Requests and downloads the Partition Information Table directly from the device.
- Supports multiple backends including `nusb` (cross-platform), `rusb` (libusb), and `serialport` (VCOM).
- Processes TAR packages in-memory and streams LZ4-compressed files directly to the device.
- Simulates the stateful Loke protocol with a mock backend for offline testing.

## Usage

### Establishing a Session with a Connected Device

```rust
use samloader_odin::{OdinManager, UsbBackendOption, create_backend};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 1. Establish the USB/VCOM backend connection
    let usb = create_backend(UsbBackendOption::Nusb, true, true)?;
    let mut odin_manager = OdinManager::new(usb, true);

    // 2. Run the Loke protocol handshake
    odin_manager.init()?;
    
    // 3. Negotiate features (packet size, LZ4 compression, etc.)
    odin_manager.begin_session()?;

    // 4. Download and print the device's PIT layout
    let pit_bytes = odin_manager.download_pit_file()?;
    println!("PIT downloaded successfully ({} bytes).", pit_bytes.len());

    // 5. Safely end the session
    odin_manager.end_session()?;

    Ok(())
}
```

## License

This project is licensed under the Apache License, Version 2.0. The core flashing logic is a derivative work of [Grimler/Heimdall](https://git.sr.ht/~grimler/Heimdall) licensed under the MIT License.
