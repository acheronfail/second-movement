# Building your Sensor Watch firmware

This checkout targets the **Sensor Watch Classic green board**, its temperature
sensor accessory, and the **original Casio LCD**. The temperature sensor is
detected automatically. If you installed the replacement custom LCD, use
`DISPLAY=custom` instead.

## Build

From this directory:

```sh
make
```

The uploadable file is `build/firmware.uf2`. This builds the custom face
selection in `movement_config.h`.

Main rotation: Clock, World Clock 2, Sunrise/Sunset, Moon Phase, Fast Stopwatch,
Timer, Advanced Alarm, Days Since, Ish, Tally, Pulsometer.

Secondary list (hold MODE from Clock): Temperature, Battery, Settings, Time Set.

The explicit equivalent is:

```sh
make BOARD=sensorwatch_green DISPLAY=classic
```

Run `make clean` before rebuilding after changing board, display, compiler, or
other build flags; those changes are not tracked by the upstream Makefile.

## Choose your faces

Edit the `watch_faces` array in `movement_config.h`, then run `make` again.
Available face declarations are in `movement_faces.h`; usage instructions
usually appear in each face's header under `watch-faces/`.

For example:

- Add `temperature_logging_face,` after `temperature_display_face,` for an
  hourly temperature history covering 36 hours.
- Add `interval_face,` or `tomato_face,` for interval or Pomodoro timers.

`MOVEMENT_SECONDARY_FACE_INDEX` is the zero-based index of the first face
accessed by holding MODE on the clock. Update it when changing the list.
The current expression hides the last four faces, starting with temperature
display. Adding faces to the main rotation keeps that boundary intact. If
adding temperature logging to the secondary list, change the expression to
`MOVEMENT_NUM_FACES - 5` so temperature display remains its first face.

Firmware must fit the watch's memory; the linker reports an error if it does not.

## Upload

1. Connect the board using a USB data cable.
2. Double-press its reset button; the `WATCHBOOT` drive should appear.
3. Drag `build/firmware.uf2` onto `WATCHBOOT`.

Alternatively, after building and connecting the watch in bootloader mode:

```sh
make install
```

Building does not upload anything. Verify the board/display choice before copying
the firmware. Hardware behaviour still needs checking on the watch.

## Dependencies and upstream

This repository was cloned using `gh repo clone joeycastillo/second-movement . -- --recurse-submodules`.
The personal fork at `acheronfail/second-movement` is `origin`; the original
`joeycastillo/second-movement` repository is `upstream`.

Required tools: GNU Arm Embedded Toolchain (including newlib), GNU Make,
Python 3, and Git. Install the compiler with Homebrew in an interactive terminal:

```sh
brew install --cask gcc-arm-embedded
```

Enter your Mac administrator password when the package installer requests it.
Run `brew` as your normal user, not with `sudo`; Homebrew invokes the installer
with the required privileges itself. The previous installation attempt downloaded
the package but failed because it could not prompt for that password.

Verify the installation, then rebuild:

```sh
arm-none-eabi-gcc --version
make clean
make
```

Homebrew links the compiler into its `bin` directory, so an existing working
Homebrew PATH should be sufficient.

After updating upstream, sync its pinned dependencies with:

```sh
git submodule update --init --recursive
```

## Build verification

Built successfully on macOS ARM64 with Arm GNU Toolchain 15.3.rel1 from
upstream commit `4a580ee` plus the custom face configuration. The custom UF2 is
283,648 bytes; the ELF reports 139,224 bytes of text, 2,512 bytes of initialized
data, and 4,644 bytes of BSS.
Upstream compiler warnings remain. The UF2 block structure and application
start address (`0x2000`) were verified. The firmware has not been flashed or
tested on physical hardware as part of this setup.
