# Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Porting to other Cortex Devices](#porting-to-other-cortex-devices)
- [Setting Project Name](#setting-project-name)
- [Building](#building)
- [Flashing](#flashing)
- [Cleaning](#cleaning)
- [Debugging](#debugging)
    - [Session 1](#session-1)
    - [Session 2](#session-2)
    - [Frequently Used GDB Commands](#frequently-used-gdb-commands)
- [Tools](#tools)
- [References](#references)

# Overview
This project contains multiple demonstrations for board equpped with ARM Cortex MCU:

| Property             | Value                                    |
| -------------------- | ---------------------------------------- |
| Development board    | STM32 Nucleo-64 w/ embedded ST-LINK/v2-1 |
| MCU                  | STM32L476RG                              |
| MCU series           | STM32L4                                  |
| MCU CPU core         | ARM Cortex-M4                            |
| MCU CPU architecture | ARMv7E-M                                 |
| Toolchain            | gcc                                      |

The project's source code is divided into two major directories: 
- The `CMSIS` directory contains header files and source code provided by [ARM](https://github.com/ARM-software/CMSIS_6/tree/main/CMSIS/Core) and [STMicroelectronics](https://github.com/STMicroelectronics/cmsis_device_l4) that are conforming to [CMSIS 6](https://arm-software.github.io/CMSIS_6/latest/General/index.html). 
- The `User` directory contains header files and source code for the demonstractions and C runtime startup code (`crt0.S`).

# Prerequisites
Install the following packages:
- `gcc-arm-none-eabi`
- `binutils-arm-none-eabi`
- `libnewlib-arm-none-eabi`
- `gdb-multiarch`
- `cmake`
- `make`
- `openocd`

# Porting to other Cortex Devices
Based on the properties of the target device, replace the following files:

| File                   | Property               | Source |
| ---------------------- | ---------------------- | ------ |
| `core_cm4.h`           | MCU CPU core           | ARM    |
| `armv7m_mpu.h`         | MCU CPU architecture   | ARM    |
| `cmsis_gcc.h`          | Toolchain              | ARM    |
| `cmsis_gcc_m.h`        | Toolchain & MCU core   | ARM    |
| `stm32l4xx_gcc.ld`[^1] | Toolchain & MCU series | ST     |
| `stm32l476xx.h`        | MCU                    | ST     |
| `stm32l4xx.h`          | MCU series             | ST     |
| `startup_stm32l4xx.c`  | MCU series & Toolchain | ST     |
| `system_stm32l4xx.c`   | MCU series             | ST     |
| `STM32L476.svd`        | MCU                    | ST     |

> [!NOTE]
> You need to modify the compile definitions, compile options and link options in each `CMakeLists.txt` based on the properties of the target device.

[^1]: You may need to modify the linker script using correct FLASH and RAM size.

# Setting Project Name
In the project root directory, modify the `<PROJECT_NAME>` entry in `CMakeLists.txt`:
```cmake
project(
    <PROJECT_NAME>
    LANGUAGES C ASM
)
```

# Building
In the project root directory, use one of the following commands to build the firmware:
- For debug configuration:
```bash
cmake -D CMAKE_BUILD_TYPE=Debug -B build .
```
- For release configuration:
```bash
cmake -D CMAKE_BUILD_TYPE=Release -B build .
```

In the project root directory, use the following command:
```bash
cmake --build ./build
```

# Flashing
In the `build` directory, use the following command:
```bash
openocd -f interface/stlink.cfg -f target/stm32l4x.cfg -c "program User/firmware.elf verify reset exit"
```
> [!NOTE]
> When porting this project to other device, select appropriate interface and target to replace `interface/stlink.cfg` and `target/stm32l4x.cfg`.

# Cleaning
In the `build` directory, use the following command:
```bash
rm -rf *
```

# Debugging
The debugging process requires two shell sessions.

## Session 1
In any directory, use the following command:
```bash
openocd -f interface/stlink.cfg -f target/stm32l4x.cfg -c "gdb_port 3333"
```
> [!NOTE]
> When porting this project to other device, select appropriate interface and target to replace `interface/stlink.cfg` and `target/stm32l4x.cfg`.

## Session 2
In the `build` directory, open another terminal, use the following command:
```bash
gdb-multiarch User/firmware.elf
```

After seeing the prompt from GDB, use the following command:
```bash
target remote localhost:3333
```

## Frequently Used GDB Commands
| Command              | Description                            |
| -------------------- | -------------------------------------- |
| `monitor reset halt` | Reset and halt                         |
| `i b`                | List all breakpoints                   |
| `d 2`                | Delete breakpoint 2                    |
| `b foo.c:18`         | Set a breakpoint at line 18 of `foo.c` |
| `p foo`              | Print `foo` in decimal format          |
| `p/t foo`            | Print `foo` in binary format           |
| `p/x foo`            | Print `foo` in hexadecimal format      |
| `c`                  | Continue                               |
| `s`                  | Step into                              |
| `n`                  | Step over                              |
| `bt`                 | Print trace of all frames              |

# Tools
Use the following command to shows all the predefined macros
```bash
arm-none-eabi-gcc -mthumb -mcpu=cortex-m4 -mfpu=fpv4-sp-d16 -E -dM -< /dev/null | sort
```

In the `build` directory, use the following command to view the dieassembly code:
```bash
arm-none-eabi-objdump -d User/firmware.elf
```

In the `build` directory, use the following command to view the symbol table:
```bash
arm-none-eabi-objdump -t User/firmware.elf | sort
```

In the `build` directory, use the following command to view the output information:
```bash
arm-none-eabi-readelf -S User/firmware.elf
```

In the `build` directory, use the following command to view the size of the executable:
```bash
arm-none-eabi-size User/firmware.elf
```

# References
- [ARMv7-M Architecture Reference Manual (DDI 0403)](https://developer.arm.com/documentation/ddi0403/latest/)
- [Cortex-M4 Devices Generic User Guide (DUI 0553)](https://developer.arm.com/documentation/dui0553/latest/)
- [STM32L47xxx Reference Manual (RM0351)](https://www.st.com/resource/en/reference_manual/rm0351-stm32l47xxx-stm32l48xxx-stm32l49xxx-and-stm32l4axxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
- [STM32L476xx Datasheet (DS10198)](https://www.st.com/resource/en/datasheet/stm32l476je.pdf)
- [STM32 Nucleo-64 Boards User Manual (UM1724)](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf)