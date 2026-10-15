# classicmini-cross-toolchain

Shared ARM/armhf cross-compile Docker toolchain for NES/SNES Classic
(hakchi/canoe) homebrew mods, built with a GCC 4.9.4/glibc 2.21 toolchain
matching the console's own runtime (GLIBC <= 2.23, GLIBCXX <= 3.4.20), plus
the console's SDL2, libpng16, ALSA, FreeType and SDL2_ttf, and a pinned UPX
build for compressing the resulting binaries.

Every library is built from public source during `docker build`; nothing is
copied off a console, so the image can be built (or pulled) by anyone.

Originally built for [OptionsMenu](https://github.com/DefKorns/OptionsMenu),
extracted here so every hmod that needs the same toolchain (OptionsMenu,
om_theme-selector, pngpack, ...) can reference one shared, versioned copy
instead of maintaining its own.

## Usage

Add as a git submodule in the consuming project (conventionally at
`toolchain/`):

```sh
git submodule add https://github.com/DefKorns/classicmini-cross-toolchain.git toolchain
```

Build the image with this repo's root as the build context:

```sh
docker build -f toolchain/Dockerfile.jessie-armhf -t <image-tag> toolchain/
```

Then run your own project's `Makefile.docker` inside it, e.g.:

```sh
docker run --rm -v "$PWD:/src" <image-tag> make -f Makefile.docker CROSS_PREFIX=arm-linux-gnueabihf-
```

Each consuming project keeps its own `Makefile.docker` with its own build
targets - this image only provides the compiler, libraries, and `git`/`rsync`
(so a `Makefile.docker` target can tag versions and package a `.hmod` from
inside the container, not just compile).

## Contents

- `Dockerfile.jessie-armhf` - the toolchain image definition.

## Libraries

Versions follow the older of the two consoles (NES Classic), so nothing links
against a symbol one of them lacks. Sources are pinned by sha256.

| Library | Version | Source |
| --- | --- | --- |
| SDL2 | 2.0.4, Nintendo's fork (Mali video driver) | [snes-classic-oss](https://github.com/DefKorns/snes-classic-oss) mirror of Nintendo's OSS release, pinned commit |
| alsa-lib | 1.0.29, unversioned symbols | alsa-project.org |
| libpng | 1.6.20 | sourceforge.net/projects/libpng |
| FreeType | 2.8.1, no zlib/bzip2/png/harfbuzz | savannah.gnu.org |
| SDL2_ttf | 2.0.14 | libsdl.org |

SDL2 links against generated stubs of the Mali GPU driver (`libMali`,
`libEGL`, `libGLESv2`, `libGLESv1_CM`); the console's own driver is used at
runtime. Its exported symbols and `NEEDED` entries match the console's
`libSDL2.so`. Consumers link with `-Wl,--allow-shlib-undefined`, since the
driver libraries aren't in the image.

FreeType and SDL2_ttf aren't on the console's rootfs, so hmods that use them
ship the image's copies (`libfreetype.so.6`, `libSDL2_ttf-2.0.so.0`).

## Debug tools (not part of the build)

- `vendor/libSegFault.so` (gitignored, not in this repo) - the console's own
  glibc SegFault helper. Copy it onto the console and run a crashing binary
  with `LD_PRELOAD=/path/to/libSegFault.so ./binary` to get a backtrace on
  segfault instead of a silent crash. No recompilation needed - works with
  any already-built binary. (AddressSanitizer is also present in the
  console's library dump, but isn't vendored here: it needs `-fsanitize=address`
  at compile time, and its ~3-4x memory overhead is a real risk on the
  Classic Mini's limited RAM.)
  