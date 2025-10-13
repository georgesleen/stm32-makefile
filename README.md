ChatGPT was integral to this project... and documentation...

# Makefile Helper for STM32CubeIDE Projects

Free yourself from the tyranny of flashing code via the STM32CubeIDE GUI.

## Why?

- **You don't have to use STM32CubeIDE** – work in Helix, Vim, VSCode, etc. without having to touch CubeIDE beyond pin assignment.
- **Compatible** – the helper invokes CubeIDE's auto-generated `Debug/makefile`, so builds match the IDE exactly.
- **Clangd/LSIF friendly** – the `compile_commands` target uses `bear` to regenerate `compile_commands.json` so language servers keep current include paths.
- **OpenOCD integration** – automatically locates CubeIDE's bundled OpenOCD script folder (`st_scripts`) so SWV/GDB settings work just like the IDE.

## Default assumptions

The helper assumes the standard CubeIDE layout:

- Build configuration folder: `Debug/`
- Artifact name equals the project directory (`wingsail-controller` → `Debug/wingsail-controller.elf`)
- OpenOCD configuration file at `<project>/<project>.cfg`

If your project uses a different configuration (e.g. `Release/`, custom target name, alternative `.cfg`), override the variables when invoking make (see below).

## Usage

Run all commands inside the project directory unless you override `PROJECT_ROOT`.

```bash
make             # print available targets and options
make build       # build (equivalent to CubeIDE build)
make flash       # build + program the MCU via OpenOCD
make gdb         # build + launch arm-none-eabi-gdb attached to open GDB server
make debug-server
make compile_commands  # regenerate compile_commands.json with bear
make clean
```

### Quiet mode

To hide command chatter and only show warnings/errors:

```bash
make build QUIET=1
a.k.a. make -s build
```

Quiet mode pipes the CubeIDE build output through a light filter that removes "Finished building…" and size summaries.

### Overriding paths and names

You can customise everything without editing the file. Pass overrides on the command line or set them in your shell. For example:

```bash
# Build the Release configuration
make build BUILD_CONFIG=Release

# Build and flash a custom-named firmware from a different project root
make flash PROJECT_ROOT=/path/to/project TARGET_NAME=my-controller

# Use a different OpenOCD config and script folder
make flash OPENOCD_CFG=/path/to/board.cfg \
           OPENOCD_SCRIPT_DIR=/opt/openocd/scripts
```

`make` will still sanitise the unsupported `-fcyclomatic-complexity` flag before invoking the Cube-generated makefiles, so you can swap build folders freely.

### compile_commands.json for clangd

`make compile_commands` runs `bear --output <BUILD_DIR>/compile_commands.json` and then symlinks it back to the project root. Re-run this target whenever CubeIDE regenerates the project and you want fresh flags for clangd/Helix.

## Reusing the helper in other projects

1. Copy this `Makefile` into the root of another CubeIDE-generated project.
2. Optionally rename firmware artifacts or build folders via variables (`TARGET_NAME`, `BUILD_CONFIG`).
3. Run `make build` / `make flash` as shown above.
