# FDxAVRC IDE-Less 1.12

**Portable Windows build, report, upload, and verification script for 8-bit AVR C, C++, and GNU assembly projects.**

<img align="left" src="IDEless-Pig.png" width="600px" style="margin-right: 20px;" />

`Z-FDxAVRC_IDE-Less_1.12.bat` provides a complete AVR-GCC and AVRDUDE workflow without requiring an IDE, project generator, Makefile, or CMake project. Put the script in a project folder, edit the settings near the top, and build by double-clicking it or running an action from Command Prompt.

## What it does

- Builds application or bootloader firmware.
- Supports `.c`, `.cc`, `.cpp`, `.cxx`, `.s`, and `.S` source files.
- Supports pure C, pure C++, mixed C/C++, and GNU assembly projects.
- Recursively scans selected source folders.
- Excludes generated, archived, or unrelated folders.
- Reuses unchanged object files for incremental builds.
- Detects changed headers through compiler dependency files.
- Selects `avr-gcc` or `avr-g++` correctly for the final link.
- Creates Flash HEX, optional EEPROM HEX, ELF, MAP, LSS, and symbol reports.
- Reports Flash, SRAM, and EEPROM use when device capacity is known.
- Builds bootloaders at validated addresses for supported classic AVR profiles.
- Uploads and verifies through AVRDUDE using USB or UART programmers.
- Resolves installed AVR-GCC MCU names and AVRDUDE part IDs.
- Prevents unverified automatic lock-byte writes.
- Shows raw compiler, linker, converter, and AVRDUDE errors once.
- Uses a fail-safe child-CMD wrapper so parser failures remain visible.

## Requirements

FDxAVRC is designed for Windows Command Prompt and requires:

1. An AVR GNU Toolchain containing `avr-gcc.exe`, `avr-g++.exe`, `avr-objcopy.exe`, and related tools.
2. AVRDUDE for upload, verification, probing, and installed-device discovery.
3. A compatible programmer or bootloader, such as USBasp, AVR109, AVR910, Butterfly, STK500v1, or STK500v2.
4. An 8-bit AVR target supported by the installed compiler.

The toolchain and AVRDUDE are **not included**.

## Quick start

### 1. Create a project folder

```text
MyFirmware\
    Z-FDxAVRC_IDE-Less_1.12.bat
    main.c
    include\
        board.h
    drivers\
        gpio.c
        uart.cpp
```

The script can remain in the project root while source files are organized in any number of subfolders.

### 2. Edit the essential settings

Open the batch file in a text editor:

```bat
set "AVR_TOOLCHAIN_BIN=C:\App\avr\avr8-gnu-toolchain-win32_x86\bin"
set "AVRDUDE_DIR=C:\App\avr\avrdude"

set "MCU=atmega88"
set "AUTO_UPLOAD=no"
set "BUILD_TYPE=application"
```

Start with `AUTO_UPLOAD=no` while checking a new project. Change it to `yes` when double-click building should also upload.

### 3. Check the configuration

```bat
Z-FDxAVRC_IDE-Less_1.12.bat check
```

The check action validates the selected MCU, compiler, AVRDUDE part, programmer path, build profile, memory information, and lock-byte safety. It also compiles and links a temporary minimal program for the selected MCU.

### 4. Build

```bat
Z-FDxAVRC_IDE-Less_1.12.bat build
```

Or double-click the script. With no command-line action, it builds and then follows `AUTO_UPLOAD`.

### 5. Upload

```bat
Z-FDxAVRC_IDE-Less_1.12.bat build-upload
```

## Command actions

| Action | Purpose |
|---|---|
| No argument | Build, then follow `AUTO_UPLOAD`. |
| `build` | Incremental build without uploading. |
| `rebuild` | Clean the selected app or boot profile and rebuild everything. |
| `clean` | Delete the selected app or boot output. |
| `clean-all` | Delete the complete configured output folder. |
| `upload` | Upload the existing firmware images without rebuilding. |
| `build-upload` | Build, then upload regardless of `AUTO_UPLOAD`. |
| `verify` | Verify existing Flash and selected EEPROM images. |
| `probe` | Connect with AVRDUDE and display device information. |
| `size` | Display size information from the existing ELF. |
| `lss` | Regenerate the source/disassembly listing from the ELF. |
| `symbols` | Regenerate the symbol-size report from the ELF. |
| `check` | Validate and explain the current configuration. |
| `list-mcus` | List MCU names installed with AVR-GCC. |
| `list-parts` | List AVRDUDE part IDs. |
| `list-programmers` | List AVRDUDE programmer IDs. |
| `help` | Display command help. |

Examples:

```bat
Z-FDxAVRC_IDE-Less_1.12.bat probe
Z-FDxAVRC_IDE-Less_1.12.bat list-mcus
Z-FDxAVRC_IDE-Less_1.12.bat verify
```

## Source discovery

```bat
:: Source folders scanned recursively; [.] scans the whole project.
set "SOURCE_DIRS=."

:: Additional excluded folders.
set "EXCLUDE_DIRS=.git;.vs"
```

Common layouts:

```bat
:: Scan the entire project folder and all subfolders.
set "SOURCE_DIRS=."

:: Scan only selected source trees.
set "SOURCE_DIRS=src;common;drivers"
```

`SOURCE_DIRS=.` already includes its subfolders. Using `.;src` is redundant, but the script normalizes and suppresses duplicate source paths.

Supported source extensions:

| Extension | Tool |
|---|---|
| `.c` | `avr-gcc` |
| `.cc`, `.cpp`, `.cxx` | `avr-g++` |
| `.s` | GNU assembler source through the compiler driver |
| `.S` | Preprocessed GNU assembler source through the compiler driver |

## C, C++, and mixed projects

```bat
:: Final linker: [auto, c, cpp].
set "PROJECT_LANGUAGE=auto"
```

Source extensions always select their own compiler. `PROJECT_LANGUAGE` controls only the final linker driver:

- `auto` uses `avr-g++` when any C++ source is present.
- `c` forces the C linker driver.
- `cpp` forces the C++ linker driver.

For C functions included from C++ source, use normal `extern "C"` guards in shared headers.

## Compiler configuration

```bat
set "COMMON_COMPILE_OPTIONS=-Os -Wall -ffunction-sections -fdata-sections"
set "C_COMPILE_OPTIONS="
set "CPP_COMPILE_OPTIONS="
set "ASM_COMPILE_OPTIONS="
set "LINK_OPTIONS=-Wl,--gc-sections"
```

Useful AVR options include:

| Option | Purpose |
|---|---|
| `-Os` | Optimize for code size. |
| `-Og -g` | Debug-friendly optimization with debug information. |
| `-O2` | Stronger speed optimization. |
| `-Wall -Wextra` | Enable useful warning groups. |
| `-ffunction-sections -fdata-sections` | Place functions and data in separate sections. |
| `-Wl,--gc-sections` | Remove unused sections during linking. |
| `-Wl,--relax` | Relax eligible calls and branches. |
| `-flto` | Enable link-time optimization. |
| `-fno-exceptions -fno-rtti` | Reduce C++ support overhead when the project permits. |

Blank `C_STANDARD` and `CPP_STANDARD` values use the installed compiler defaults, which is useful with older AVR toolchains.

## Definitions and include folders

```bat
:: Definitions without -D.
set "DEFINES=DEBUG;USB_CFG=1"

:: Header folders without -I.
set "INCLUDE_DIRS=include;lib\include"
```

Semicolon-separated settings preserve spaces inside individual paths and are converted to the proper compiler options by the script.

## `F_CPU`

The preferred approach is to define the clock in source code or a shared project header:

```c
#define F_CPU 16000000UL
```

The batch setting is available when a command-line definition is preferred:

```bat
set "F_CPU=16000000UL"
```

When `F_CPU` is set in the batch file, it is passed to every compilation as `-DF_CPU=...`. When blank, the script searches for a simple source `#define F_CPU` for display and filename generation. Conflicting source definitions produce a warning instead of silently selecting one.

## Application builds

```bat
set "BUILD_TYPE=application"
```

Application mode uses normal startup files and normal program placement.

Optional application-only settings:

```bat
set "APPLICATION_COMPILE_OPTIONS="
set "APPLICATION_LINK_OPTIONS="
```

A custom linker section can be placed at a fixed address:

```bat
set "CUSTOM_SECTION_NAME=FD_App_Start_Add"
set "CUSTOM_SECTION_ADDRESS=0x0"
```

Set `CUSTOM_SECTION_NAME=` blank to disable custom section placement.

## Bootloader builds

```bat
set "BUILD_TYPE=bootloader"
set "BOOTLOADER_SIZE_BYTES=1024"
set "BOOT_START_ADDRESS=auto"
```

For verified classic AVR profiles, `auto` calculates:

```text
Boot start = Flash capacity - reserved bootloader size
```

The script checks whether the selected profile supports the requested boot size before applying automatic placement.

Important:

- The MCU's `BOOTSZ` and `BOOTRST` fuse settings must match the compiled bootloader address.
- FDxAVRC does not automatically program those fuses.
- Startup files are kept by default.
- `BOOT_NO_STARTFILES=yes` is an advanced option, not a normal requirement.
- Bootloader uploads request write protection only for verified classic lock layouts.

## Output files

Application and bootloader build files are separated:

```text
_Output\
    app\
        project.elf
        project.map
        project.lss.txt
        objects and dependencies...

    boot\
        project.elf
        project.map
        project.lss.txt
        objects and dependencies...

    APP_MyFirmware_16MHz_Atmega88.hex
    APP_MyFirmware_16MHz_Atmega88.EEPROM.hex
    BOOT_MyFirmware_16MHz_Atmega88.hex
```

The exact clock tag is included only when a usable `F_CPU` value is available.

| File | Purpose |
|---|---|
| `.hex` | Intel HEX image for program Flash. |
| `.EEPROM.hex` | Initialized EEPROM image; created only when `.eeprom` contains data. |
| `.elf` | Linked image with sections, symbols, and debug information. |
| `.lss.txt` | Mixed source and disassembly listing. |
| `.map` | Linker section, symbol, object, and library map. |
| `.symbols.txt` | Optional symbol-size report generated by `avr-nm`. |

Runtime EEPROM calls such as `eeprom_write_byte()` do not create an EEPROM image by themselves. An initialized `EEMEM` object does.

## Firmware-size report

When device capacity is available, the build displays section sizes and percentage use:

```text
Firmware size:
-----------------------------------------------
text    data     bss     dec     hex
3562      41       4    3607     e17

Flash:  3603    bytes  / 8192    bytes  43.9%
SRAM :  45      bytes  / 1024    bytes  4.3%
EEPROM: 3       bytes  / 512     bytes  0.5%
```

Built-in profiles supply common capacities. Unknown compiler-supported MCUs can still build; manual capacities are needed only for percentage reporting or automatic bootloader calculations.

## Programmer configuration

### USB programmer

```bat
set "PROGRAMMER_CONNECTION=usb"
set "PROGRAMMER=usbasp"
set "AVRDUDE_OPTIONS=-B 1.0"
```

The USB path omits AVRDUDE port and baud options.

### UART programmer or bootloader

```bat
set "PROGRAMMER_CONNECTION=uart"
set "PROGRAMMER=avr109"
set "PROGRAMMER_PORT=COM3"
set "PROGRAMMER_BAUD=115200"
```

The UART path adds the configured `-P` and `-b` values.

Other common AVRDUDE programmer IDs include:

```text
butterfly, avr109, avr910, stk500v2, stk500v1
```

Use `list-programmers` to see the IDs supported by the installed AVRDUDE.

## Flash and EEPROM upload

```bat
:: EEPROM upload: [no, yes, auto]
set "UPLOAD_EEPROM=no"
```

- `no` never alters EEPROM.
- `yes` requires and uploads the current `.EEPROM.hex` image.
- `auto` uploads EEPROM only when the current image exists.

The separate `verify` action follows the same EEPROM selection.

## Lock-byte safety

```bat
set "APPLICATION_LOCK_MODE=unlocked"
```

Available application modes:

| Mode | Behavior |
|---|---|
| `unlocked` | Performs no lock-byte write. |
| `bootProtect` | Writes verified classic boot-section protection. |
| `appProtect` | Writes verified classic application-section protection. |
| `fullLock` | Writes the verified classic full-lock value. |

This setting controls the lock byte only. It does not configure clock, brown-out, boot-size, reset-vector, or other fuse settings.

Unknown, XMEGA, and newer families do not receive a guessed lock value. The script stops before an unverified automatic lock write.

## Built-in MCU profiles

The script includes profile information for common classic devices, including:

```text
ATmega8, ATmega48P, ATmega88, ATmega88P, ATmega168, ATmega168P,
ATmega328P, ATmega16, ATmega32, ATmega64, ATmega128, ATmega169P,
ATmega644P, ATmega1284P, ATtiny10, ATtiny13, ATtiny13A,
ATtiny24, ATtiny44, ATtiny84, ATtiny25, ATtiny45, ATtiny85,
and ATtiny2313
```

A built-in profile adds memory capacity, common AVRDUDE mapping, and—where verified—automatic bootloader and lock-layout metadata.

The compiler remains the authority for build support. Run:

```bat
Z-FDxAVRC_IDE-Less_1.12.bat list-mcus
```

Any exact MCU name accepted by the installed AVR-GCC can be used for compilation. `AVRDUDE_PART=auto` retrieves an existing part ID from the installed AVRDUDE list; it does not invent one from a naming formula.

## Incremental builds

With:

```bat
set "RECOMPILE_ALL=no"
```

an object is reused when its source and dependency headers have not changed. The build signature includes important configuration such as:

- Compiler and tool paths.
- MCU and firmware type.
- Clock definition.
- Language standards.
- Compile and linker options.
- Include and library paths.
- Bootloader placement.
- Custom section settings.
- Linked objects and libraries.

Changing the MCU or another relevant setting forces a correct full rebuild. Use `rebuild` when a deliberate clean compile is required.

## Generated reports

```bat
set "KEEP_ELF=yes"
set "CREATE_LSS=yes"
set "CREATE_MAP_FILE=yes"
set "CREATE_EEPROM_FILE=yes"
set "CREATE_SYMBOL_REPORT=no"
```

Reports can also be regenerated from an existing ELF:

```bat
Z-FDxAVRC_IDE-Less_1.12.bat size
Z-FDxAVRC_IDE-Less_1.12.bat lss
Z-FDxAVRC_IDE-Less_1.12.bat symbols
```

Each action validates only the tools it actually needs. For example, `probe` requires AVRDUDE but does not require the compiler.

## Suggested project layouts

### Small project

```text
Blink\
    Z-FDxAVRC_IDE-Less_1.12.bat
    main.c
```

### Structured project

```text
Controller\
    Z-FDxAVRC_IDE-Less_1.12.bat
    src\
        main.cpp
        application.cpp
    drivers\
        gpio.c
        timer.c
        uart.c
    include\
        board.h
        application.hpp
    lib\
        custom_driver.a
    _Output\
```

Suggested settings:

```bat
set "SOURCE_DIRS=src;drivers"
set "INCLUDE_DIRS=include"
set "LIBRARY_DIRS=lib"
set "LIBRARIES=custom_driver;m"
```

## Troubleshooting

### The selected MCU does not compile

Run:

```bat
Z-FDxAVRC_IDE-Less_1.12.bat list-mcus
```

Use the exact name supported by the installed toolchain.

### AVRDUDE cannot resolve the device

Run:

```bat
Z-FDxAVRC_IDE-Less_1.12.bat list-parts
```

Leave `AVRDUDE_PART=auto` when the installed AVRDUDE list contains a matching full device name. Otherwise, set the exact part ID manually.

### USBasp communication is too fast

Try a slower ISP period:

```bat
set "AVRDUDE_OPTIONS=-B 4"
```

The required value depends on the target clock and programmer firmware.

### A C function is undefined from C++

Place shared C declarations inside `extern "C"` guards when included by C++.

### The EEPROM file is missing

The file is generated only when the linked ELF contains initialized EEPROM data. Runtime EEPROM access alone is not an initializer.

### A bootloader build is rejected

The requested MCU or boot size may not have verified automatic placement metadata. Use a supported profile/size or provide a verified manual `BOOT_START_ADDRESS`.

### Old objects appear to be reused incorrectly

Run:

```bat
Z-FDxAVRC_IDE-Less_1.12.bat rebuild
```

A normal MCU or configuration change should already invalidate the object cache through the build signature.

## Design goals

FDxAVRC is intentionally a single readable batch file. It keeps the build process visible and editable while avoiding the overhead of an IDE project format.

It does not attempt to:

- Download or install toolchains.
- Hide compiler and linker behavior.
- Guess dangerous fuse or lock settings.
- Replace the MCU datasheet.
- Replace AVR-GCC, AVR-LibC, or AVRDUDE.
- Become a general project-management system.

The script is intended to make the normal AVR toolchain easier to use, inspect, learn, and adapt.

## Buy Me a Coffee

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://paypal.me/flyandance?country.x=US&locale.x=en_US)
