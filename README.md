# NVRAMHW - NVRAM Vector Hardware Module

A RISC OS module that implements the NVRAMV software vector for non-volatile RAM operations.

## Overview

This module provides a prototype implementation of the NVRAMV vector (vector 0x3E), which is used by RISC OS Kernel 9.48 and later to delegate NVRAM handling to hardware support modules. The module intercepts NVRAM operations and provides stub implementations that simulate NVRAM hardware.

## Features

- **NVRAMV Vector Implementation**: Claims and handles the NVRAMV vector
- **Three Operations Supported**:
  - **FillCache** (reason 0): Populates a cache buffer with NVRAM data (up to 240 bytes)
  - **ReadByte** (reason 1): Reads a single byte from NVRAM
  - **WriteByte** (reason 2): Writes a single byte to NVRAM
- **Stub Hardware Implementation**: Simulates NVRAM with in-memory storage
- **Debug Output**: Comprehensive debug logging of all operations
- **PCF8583 Example**: Comments describing how to interface with a real PCF8583 RTC/NVRAM chip via IIC

## Requirements

- RISC OS 3.1 or later
- AMU (RISC OS Make Utility) build environment
- CMHG (C Module Header Generator)

## Building

### Build the Module

```bash
riscos-amu
```

This produces `rm32/NVRAMHW` (32-bit RAM module).

### Build All Variants

```bash
# 32-bit RAM module
riscos-amu ram

# 26-bit RAM module
riscos-amu ram BUILD26=1 BUILD32=

# 32-bit ROM module
riscos-amu rom

# 26-bit ROM module
riscos-amu rom BUILD26=1 BUILD32=

# Export headers
riscos-amu export
```

### Clean Build

```bash
riscos-amu clean
riscos-amu
```

## Usage

### Loading the Module

```basic
*RMLoad rm32.NVRAMHW
```

### Unloading the Module

```basic
*RMKill NVRAMHW
```

### Testing

Run the included test program:

```basic
*RMLoad rm32.NVRAMHW
*Run TestNVRAM,fd1
```

The test program performs 10 tests:
1. Module loaded verification
2. ReadByte at address 0
3. ReadByte at address 1
4. ReadByte at addresses 2-5
5. ReadByte out of range (address 255)
6. WriteByte and ReadByte verification
7. WriteByte value preservation
8. FillCache operation
9. Multiple reads consistency
10. WriteByte out of range handling

## Technical Details

### Vector Interface

The NVRAMV vector is called by the Kernel with the following register interface:

| Reason | Name      | Entry (R0-R2)              | Exit (R0-R2)                    |
|--------|-----------|----------------------------|---------------------------------|
| 0      | FillCache | R1=cache ptr, R2=count     | R0=-1, R2=bytes populated       |
| 1      | ReadByte  | R1=address                 | R0=-1, R1=value read            |
| 2      | WriteByte | R1=address, R2=value       | R0=-1, R1=value written         |

### NVRAM Layout

The stub implementation provides 240 bytes of usable NVRAM (addresses 0-239). The initial contents are:

| Address | Value | Description      |
|---------|-------|------------------|
| 0       | &5A   | Test pattern     |
| 1       | &A5   | Test pattern     |
| 2       | &DE   | Test pattern     |
| 3       | &AD   | Test pattern     |
| 4       | &BE   | Test pattern     |
| 5       | &EF   | Test pattern     |
| 6-239   | &00   | Zero initialised |

### Integration with Kernel

The Kernel calls NVRAMV through OS_Byte calls:
- **OS_Byte 160**: FillCache
- **OS_Byte 161**: ReadByte
- **OS_Byte 162**: WriteByte

The module must be loaded early in the boot sequence to intercept these calls before the Kernel initialises its NVRAM cache.

## Hardware Implementation Notes

The stub implementation in `c/nvram` includes comments describing how to interface with a PCF8583 RTC/NVRAM chip:

- **IIC Address**: 0xA0 (write), 0xA1 (read)
- **RAM Location**: Addresses 0x0A-0xFF (240 bytes)
- **Write Cycle Time**: ~5ms

To implement real hardware support:
1. Initialise the IIC bus
2. Detect PCF8583 presence
3. Implement read/write operations via IIC protocol
4. Handle write cycle delays

## Project Structure

```
NVRAMHW/
├── c/
│   ├── module      # Module interface (init, final, vector handler)
│   ├── nvram       # NVRAM hardware stub implementation
│   └── os          # OS interface (vector claim/release)
├── cmhg/
│   └── modhead     # CMHG module definition
├── h/
│   ├── nvram       # NVRAM hardware interface header
│   └── os          # OS interface header
├── rm32/
│   └── NVRAMHW     # Built 32-bit RAM module
├── oz32/
│   └── *.o         # Object files
├── .robuild.yaml   # RISC OS build configuration
├── Makefile,fe1    # AMU makefile
├── TestNVRAM,fd1   # BBC BASIC test program
├── VersionNum      # Version information
└── LICENSE         # MIT license
```

## CI/CD

The project includes automated build configuration:

- **`.robuild.yaml`**: RISC OS build script for build.riscos.online
- **`.github/workflows/ci.yml`**: GitHub Actions workflow
- **`.gitlab-ci.yml`**: GitLab CI configuration

Builds produce:
- 32-bit and 26-bit RAM modules
- 32-bit and 26-bit ROM modules
- Exported headers

## License

MIT License - see [LICENSE](LICENSE) for details.

Copyright (c) 2026 Charles Ferguson

## Author

**Charles Ferguson**

## Related Documentation

- [NVRAMV Vector Specification](nvramv.xml)
- [RISC OS PRM - Software Vectors](http://www.riscos.com/support/developers/prm/softvecs.html)
- [RISC OS PRM - OS_Byte](http://www.riscos.com/support/developers/prm/osbyte.html)
