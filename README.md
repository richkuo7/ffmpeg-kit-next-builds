# ffmpeg-kit-next-builds

LGPL FFmpeg builds for React Native, published with the complete recipe that
produced them.

This repository exists to satisfy the LGPL. It carries the licence texts, the exact
inputs needed to rebuild these binaries from source, and the released artifacts
themselves. It contains no proprietary code.

## Licence

The published binaries are licensed under the **GNU Lesser General Public License,
version 3**. See [`COPYING.LESSER`](COPYING.LESSER), which incorporates the GNU
General Public License version 3 in [`COPYING`](COPYING) by reference.

The build enables **no GPL component**. It carries `--enable-version3` and does not
carry `--enable-gpl`, `--enable-nonfree`, libx264, libx265, libxvid, vid.stab or
rubberband. You can confirm that yourself from the binaries — see *Verifying* below.

The FFmpeg libraries are built **shared** (`--enable-shared --disable-static`), so an
application that links them can be relinked against your own modified build by
replacing the frameworks or the `.so` files.

## Corresponding source

The source is FFmpegKitNext at a pinned commit, unmodified:

| | |
|---|---|
| Project | [`arthenica/ffmpeg-kit-next`](https://github.com/arthenica/ffmpeg-kit-next) |
| Tag | `v8.1.1` |
| Commit | `1022763a9488f0995db6505d4d4142347714758d` |
| FFmpeg | 8.1.2 (`n8.1.2`) |

FFmpegKitNext downloads and builds FFmpeg itself; its own scripts pin the FFmpeg
source. No patches of ours are applied to either project.

## Rebuilding

Install [Nix](https://nixos.org) with flakes, and Xcode 26 or newer for the Apple
build. FFmpegKitNext supplies the Android SDK and NDK through its own Nix shell.

```bash
git clone https://github.com/arthenica/ffmpeg-kit-next.git
cd ffmpeg-kit-next
git checkout --detach 1022763a9488f0995db6505d4d4142347714758d
```

Then run the two builds with exactly these flags:

```bash
./nix-ios.sh --profile xcode26 \
  --xcframework \
  --disable-arm64e --disable-arm64-mac-catalyst \
  --disable-x86-64-mac-catalyst \
  --enable-ios-videotoolbox --enable-ios-audiotoolbox \
  --enable-ios-zlib --enable-ios-bzip2 --enable-ios-libiconv \
  --enable-libass --enable-freetype --enable-fontconfig

./nix-android.sh --profile android-r27d \
  --disable-x86 --disable-arm-v7a \
  --enable-android-zlib --enable-android-media-codec \
  --enable-libiconv \
  --enable-libass --enable-freetype --enable-fontconfig

cd react-native && ./copy_local_binaries.sh
```

`--enable-libass` transitively enables freetype, fribidi, fontconfig, harfbuzz,
expat, libuuid and libiconv, and freetype in turn enables libpng and libjxl. All of
those are permissive or LGPL.

The released tarball is that `react-native/` directory, renamed to `package/` so it
installs as an npm package, with `ios/Frameworks/` and `android/libs-maven/` included.

## What the releases contain

| Platform | Architectures |
|---|---|
| iOS | device `arm64`; simulator `arm64` + `x86_64` |
| Android | `arm64-v8a`, `armeabi-v7a`, `x86_64` (API 24) |

Release tags are `v<upstream version>-<rebuild number>`, so `v8.1.1-1` is the first
published rebuild of upstream `v8.1.1`. A tag is never moved: each one names an exact
set of bytes, so any rebuild gets the next number.

The Intel simulator slice is built even though no shipped app runs on it. Without it,
Xcode on an Intel Mac cannot find the framework headers and reports a missing include,
which reads like a broken install.

## Verifying

Read the configure line out of a shipped binary rather than trusting any metadata:

```bash
# iOS
strings -a package/ios/Frameworks/libavutil.xcframework/ios-arm64/libavutil.framework/libavutil \
  | grep -m1 -o -- "--prefix=.*" | tr ' ' '\n' | sort

# Android
unzip -o package/android/libs-maven/com/arthenica/ffmpeg-kit-next/8.1.1/ffmpeg-kit-next-8.1.1.aar -d aar
strings -a aar/jni/arm64-v8a/libavutil.so | grep -m1 -o -- "--prefix=.*" | tr ' ' '\n' | sort
```

A build is not bit-reproducible: FFmpeg embeds that configure line, and it carries
the build directory, the SDK path and a build-date stamp. What reproduces is the
content — the pinned commit plus the flag list above.

## Trademark and copyright

FFmpeg is copyright of the FFmpeg developers. This repository is not affiliated with
the FFmpeg project or with ARTHENICA.

---
Created with LLM: Opus 5 | high | Harness: Claude Code
