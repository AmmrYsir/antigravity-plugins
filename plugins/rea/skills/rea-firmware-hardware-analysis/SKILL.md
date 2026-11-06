---
name: rea-firmware-hardware-analysis
description: Embedded systems, firmware, WebAssembly, and hardware signal reverse engineering with REA. Unpack firmware images via Binwalk/Unblob, disassemble WebAssembly (WASM) via WABT, recover EVM smart contract interfaces, and decode logic traces via sigrok.
---

# Firmware & Hardware Signal Analysis with REA

This skill provides specialized procedures for IoT firmware images, smart contract bytecode, compiled WebAssembly binaries, and digital hardware signal recordings.

---

## 1. Embedded Firmware Image Analysis

REA provides tools to inspect and carve flash ROM dumps, router firmware images, and microcontroller updates.

### Inspect Firmware Regions
Identify embedded filesystems (SquashFS, CramFS, JFFS2, UBIFS), bootloaders (U-Boot), and compressed payloads before carving:
```json
{
  "path": "/path/to/firmware.bin"
}
```
- Tool: `inspect_firmware_regions`
- Leverages Binwalk / Unblob to detect magic headers, compression schemes (LZMA, Gzip, Zstandard), and partition boundaries.

### Extract Firmware Filesystem
Carve filesystems into a designated destination directory:
```json
{
  "path": "/path/to/firmware.bin",
  "output_directory": "/path/to/extracted_firmware"
}
```
- Tool: `extract_firmware`
- Extracted root filesystems can subsequently be analyzed for configuration scripts, default credentials, and compiled ELF binaries (dispatching to `rea-native-binary-analysis`).

---

## 2. WebAssembly (WASM) Disassembly

For WebAssembly modules found in web applications, Node.js packages, or standalone WASI runtimes:

```json
{
  "path": "/path/to/module.wasm"
}
```
- Tool: `inspect_wasm_artifact`
- **Capabilities**:
  - Decodes binary WASM bytecode into WebAssembly Text format (WAT) via WABT (`wasm2wat`).
  - Enumerates exported and imported functions, tables, and memory definitions.
  - Correlates JavaScript glue code (Emscripten / wasm-bindgen) to native WASM function indices.
  - Guarantees zero execution of WASM instructions during inspection.

---

## 3. EVM Smart Contract Bytecode Recovery

Reverse engineer Ethereum / EVM bytecode without source code:

```json
{
  "bytecode": "0x608060405234801561001057600080fd5b50...",
  "encoding": "hex",
  "detail": "summary"
}
```
- Tool: `inspect_evm_interface`
- **Output**:
  - Recovers 4-byte function selectors (e.g. `0xa9059cbb` -> `transfer(address,uint256)`).
  - Infers parameter types, return shapes, and state mutability (`view`, `pure`, `payable`, `nonpayable`).
  - Identifies fallback and receive handlers.

---

## 4. Hardware Logic Signal Decoding (Sigrok)

Decode digital logic analyzer captures (Saleae, Sigrok `.sr` files) across serial and bus protocols:

### Discover Installed Protocol Decoders
```json
{
  "query": "spi|i2c|uart|can|usb"
}
```
- Tool: `inspect_signal_decoders`
- Returns supported decoder IDs, required channel mappings (CLK, MOSI, MISO, CS), and configurable baud rates.

### Decode Recorded Signal
```json
{
  "path": "/path/to/recording.sr",
  "decoder": "spi",
  "options": {
    "clk": "0",
    "mosi": "1",
    "miso": "2",
    "cs": "3"
  },
  "line_limit": 50
}
```
- Tool: `decode_signal_recording`
- Decodes raw logic transitions into ASCII or hexadecimal byte streams, revealing microcontroller communication and sensor readouts.
