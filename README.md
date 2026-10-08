# Nyu - **Ny**vela **U**serspace Program Template

Starting point for a userspace program that runs on the Nyvela kernel.

It builds a freestanding x86-64 ELF and converts it to a flat binary that the
kernel can load directly.

## Configure

Edit `prog.conf` and fill in both variables:

```make
# Path to the 'include' folder inside Nyvela
USERLIB_PATH = ../Nyvela/include # example path

# Name of the generated binary at ./build/$(BIN_NAME).bin
BIN_NAME = hello
```

`make` errors out if either one is left blank.

## Build

```sh
make
```

Output lands in `build/`:

- `$(BIN_NAME).elf` - linked image
- `$(BIN_NAME).bin` - flat binary, objcopy'd from the ELF

Clean up with `make clean`.

## Layout

```
prog.conf   build settings (USERLIB_PATH, BIN_NAME)
makefile    compiles src/*.c, links, objcopies
linker.ld   linker script, base address 0x400000
src/        your source, entry point is _start()
include/    local headers, passed via -Iinclude
```

`_start` is the entry point declared in `linker.ld`. Nyvela jumps to the image's
link base, so `src/main.c` defines `_start` and calls into the kernel API:

```c
#include <nyvela/user/syslib.h>

void _start(void) {
  sys_exit(0);
}
```

Anything you include from `<nyvela/user/syslib.h>` is what the kernel exposes to
userspace. See Nyvela/docs for the rest of the API.

## Notes

The link base of `0x400000` matches the init program and the shell, so a program
built here can sit at the same address as those without relocating the linker
script. If you need a different base, change `. = 0x400000;` in `linker.ld` and
make sure nothing else claims that range.
