# GStreamer Rust Project

<https://gstreamer.freedesktop.org/>

## Setup

<https://docs.rs/gstreamer/latest/gstreamer/>

Install using Packages.

<https://gstreamer.freedesktop.org/data/pkg/osx/>

1.24.7 (August 21, 2024 at 8:25:15 AM GMT-3)

<https://gitlab.freedesktop.org/gstreamer/gstreamer/-/tags/1.24.7>

```bash
cd ~/Downloads

curl -kLO https://gstreamer.freedesktop.org/data/pkg/osx/1.24.7/gstreamer-1.0-1.24.7-universal.pkg

curl -kLO https://gstreamer.freedesktop.org/data/pkg/osx/1.24.7/gstreamer-1.0-devel-1.24.7-universal.pkg

installer -pkg gstreamer-1.0-1.24.7-universal.pkg -target CurrentUserHomeDirectory
installer -pkg gstreamer-1.0-devel-1.24.7-universal.pkg -target CurrentUserHomeDirectory

rm gstreamer-1.0-1.24.7-universal.pkg gstreamer-1.0-devel-1.24.7-universal.pkg

ls ~/Library/Frameworks/GStreamer.framework/Versions/1.0/bin

# ...lot of files, 3 important listed below...
# gst-launch-1.0
# gst-shell
# pkg-config


# Interative bash shell loads `.bash_profile`
# (for example, when user opens a terminal)

echo '
# >>> GStreamer >>>

case ":$PATH:" in
    *:/Users/ciro.cavani/Library/Frameworks/GStreamer.framework/Versions/1.0/bin:*)
        ;;

    *)
        export PATH=/Users/ciro.cavani/Library/Frameworks/GStreamer.framework/Versions/1.0/bin${PATH:+:${PATH}}
        ;;
esac

# <<< GStreamer <<<' \
>> ~/.bash_profile

# Non-interative bash shell loads `.bashrc`
# (for example, when rust-analyser runs by your editor)

echo '
# >>> GStreamer >>>

case ":$PATH:" in
    *:/Users/ciro.cavani/Library/Frameworks/GStreamer.framework/Versions/1.0/bin:*)
        ;;

    *)
        export PATH=/Users/ciro.cavani/Library/Frameworks/GStreamer.framework/Versions/1.0/bin${PATH:+:${PATH}}
        ;;
esac

# <<< GStreamer <<<' \
>> ~/.bashrc


source ~/.bash_profile

which gst-shell

# /Users/ciro.cavani/Library/Frameworks/GStreamer.framework/Versions/1.0/bin/gst-shell

which pkg-config

# /Users/ciro.cavani/Library/Frameworks/GStreamer.framework/Versions/1.0/bin/pkg-config
```

### Troubleshooting

#### Error `failed to run custom build command for gstreamer-sys`

Homebrew pkg-config is before GStreamer pkg-config.

Solution.

(more than one option, presenting only the one that uses GStreamer provided tool)

```sh
export PATH=${HOME}/Library/Frameworks/GStreamer.framework/Versions/1.0/bin${PATH:+:${PATH}}
```

Expected error (may be different).

```sh
cargo build
```

(Output)

```text
error: failed to run custom build command for `gstreamer-sys v0.22.2`

Caused by:
  process didn't exit successfully: `/Users/ciro.cavani/Garage/gstream-rust-project/target/debug/build/gstreamer-sys-1dbbc117f0f53ca1/build-script-build` (exit status: 1)
  --- stdout
  cargo:warning=
  pkg-config exited with status code 1
  > PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=1 pkg-config --libs --cflags gstreamer-1.0 gstreamer-1.0 >= 1.14

  The system library `gstreamer-1.0` required by crate `gstreamer-sys` was not found.
  The file `gstreamer-1.0.pc` needs to be installed and the PKG_CONFIG_PATH environment variable must contain its parent directory.
  The PKG_CONFIG_PATH environment variable is not set.

  HINT: if you have installed the library, try setting PKG_CONFIG_PATH to the directory containing `gstreamer-1.0.pc`.

warning: build failed, waiting for other jobs to finish...
```

Check environment variables.

```sh
printenv PATH
```

(Output)

```text
/opt/homebrew/bin:/opt/homebrew/sbin:/Users/ciro.cavani/Library/Frameworks/GStreamer.framework/Versions/1.0/bin:(...)
```

> _Note_
>
> When this error occurs during `rust-analyser` execution by the editor, the solution is to set your `.bashrc` to update `PATH` with GStreamer path.

## Error `Library not loaded: @rpath/libgstreamer-1.0.0.dylib`

Solution.

```sh
export DYLD_FALLBACK_LIBRARY_PATH=${HOME}/Library/Frameworks/GStreamer.framework//Versions/1.0/lib
```

Expected error (may be different).

```sh
cargo run
```

(output)

```text
(...)
Running `target/debug/gstream_rust_project`
dyld[98031]: Library not loaded: @rpath/libgstreamer-1.0.0.dylib
Referenced from: <92B5EA2A-903B-3F98-9779-F5F1388A55DF> /Users/ciro.cavani/Garage/gstream-rust-project/target/debug/gstream_rust_project
Reason: tried: '/Users/ciro.cavani/Garage/gstream-rust-project/target/debug/deps/libgstreamer-1.0.0.dylib' (no such file), '/Users/ciro.cavani/Garage/gstream-rust-project/target/debug/libgstreamer-1.0.0.dylib' (no such file), '/Users/ciro.cavani/.rustup/toolchains/stable-aarch64-apple-darwin/lib/rustlib/aarch64-apple-darwin/lib/libgstreamer-1.0.0.dylib' (no such file), '/Users/ciro.cavani/.rustup/toolchains/stable-aarch64-apple-darwin/lib/libgstreamer-1.0.0.dylib' (no such file), '/Users/ciro.cavani/lib/libgstreamer-1.0.0.dylib' (no such file), '/usr/local/lib/libgstreamer-1.0.0.dylib' (no such file), '/usr/lib/libgstreamer-1.0.0.dylib' (no such file, not in dyld cache)
Abort trap: 6
```

## Applications

<https://gitlab.freedesktop.org/gstreamer/gstreamer-rs/tree/main/tutorials>

<https://gitlab.freedesktop.org/gstreamer/gstreamer-rs/tree/main/examples>

### Camera Stream

<https://stackoverflow.com/questions/66296668/gstreamer-macos-big-sur-camera-input>

`gst-launch-1.0 avfvideosrc ! osxvideosink`

```bash
mkdir -p programmable-matter-gst
cd programmable-matter-gst

cargo init --lib
echo '' > src/lib.rs

# release: longer compilation, better binary code

echo '
[profile.release]
strip = true
lto = true
codegen-units = 1
opt-level = 3' \
>> Cargo.toml

# lib external name: `programmable_matter::*`

echo '
[lib]
name = "programmable_matter"' \
>> Cargo.toml

# Project deps:
cargo add \
anyhow \
gstreamer \
tracing \
tracing-subscriber \
--features \
tracing-subscriber/env-filter

export PATH=${HOME}/Library/Frameworks/GStreamer.framework/Versions/1.0/bin${PATH:+:${PATH}}
export DYLD_FALLBACK_LIBRARY_PATH=${HOME}/Library/Frameworks/GStreamer.framework//Versions/1.0/lib

cargo check

# Checking programmable-matter-gst v0.1.0 (/Users/ciro.cavani/Garage/programmable-matter-gst)
# Finished dev [unoptimized + debuginfo] target(s) in 11.88s



cargo run --bin camera_video
```
