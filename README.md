# hakchi-toolchain

Shared ARM/armhf cross-compile Docker toolchain for NES/SNES Classic
(hakchi/canoe) homebrew mods, built with a GCC 4.9.4/glibc 2.21 toolchain
matching the console's own runtime (GLIBC <= 2.23, GLIBCXX <= 3.4.20), plus
the console's own SDL2 headers/lib and a compatible libpng16, and a pinned
UPX build for compressing the resulting binaries.

Originally built for [OptionsMenu](https://github.com/DefKorns/OptionsMenu),
extracted here so every hmod that needs the same toolchain (OptionsMenu,
om_theme-selector, pngpack, ...) can reference one shared, versioned copy
instead of maintaining its own.

## Usage

Add as a git submodule in the consuming project (conventionally at
`toolchain/`):

```sh
git submodule add https://github.com/DefKorns/hakchi-toolchain.git toolchain
```

`vendor/sdl2-headers/` and `vendor/lib.tar` are **not** part of this repo
(see Contents below - they're extracted from the console's own firmware, not
ours to redistribute). Copy your own local copies into `toolchain/vendor/`
before building - they're gitignored, so this step is per-checkout.

Build the image with this repo's root as the build context, so `vendor/`
resolves:

```sh
docker build -f toolchain/Dockerfile.jessie-armhf -t <image-tag> toolchain/
```

Then run your own project's `Makefile.docker` inside it, e.g.:

```sh
docker run --rm -v "$PWD:/src" <image-tag> make -f Makefile.docker CROSS_PREFIX=arm-linux-gnueabihf-
```

Each consuming project keeps its own `Makefile.docker` with its own build
targets - this image only provides the compiler and libraries.

## Contents

- `Dockerfile.jessie-armhf` - the toolchain image definition.
- `vendor/sdl2-headers/`, `vendor/lib.tar` (gitignored, not in this repo) -
  the console's own SDL2 headers (2.0.4) and `libSDL2.so`, extracted from
  the console itself (generic Debian SDL2 packages don't match its
  ABI/soname). Supply your own local copies before building.
