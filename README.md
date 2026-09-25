# classicmini-cross-toolchain

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
git submodule add https://github.com/DefKorns/classicmini-cross-toolchain.git toolchain
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
targets - this image only provides the compiler, libraries, and `git`/`rsync`
(so a `Makefile.docker` target can tag versions and package a `.hmod` from
inside the container, not just compile).

## Contents

- `Dockerfile.jessie-armhf` - the toolchain image definition.
- `vendor/sdl2-headers/`, `vendor/lib.tar`, `vendor/libpng.tar`,
  `vendor/libSDL2_ttf.tar`, `vendor/libfreetype.tar`, `vendor/libasound.tar`
  (all gitignored, not in this repo) - the console's own headers/libs,
  extracted from the console itself (generic Debian packages don't match its
  exact ABI/soname). Supply your own local copies before building.

## Debug tools (not part of the build)

- `vendor/libSegFault.so` (gitignored, not in this repo) - the console's own
  glibc SegFault helper. Copy it onto the console and run a crashing binary
  with `LD_PRELOAD=/path/to/libSegFault.so ./binary` to get a backtrace on
  segfault instead of a silent crash. No recompilation needed - works with
  any already-built binary. (AddressSanitizer is also present in the
  console's library dump, but isn't vendored here: it needs `-fsanitize=address`
  at compile time, and its ~3-4x memory overhead is a real risk on the
  Classic Mini's limited RAM.)