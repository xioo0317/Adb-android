# android-sdk-tools (adb only)

English | [简体中文](README_cn.md)

Build the AOSP `adb` from the upstream source with CMake, as an **Android executable**
(statically linked, bionic libc via the Android NDK).

Based on [lzhiyong/android-sdk-tools](https://github.com/lzhiyong/android-sdk-tools) (Apache-2.0),
trimmed down to **adb only**: `repository.json` only fetches the AOSP repositories that adb needs,
and the CMake project only defines the `adb` target.

* **adb only**: `get_source.py` clones ~25 upstream AOSP repos (instead of 40+), `build.py` builds the `adb` target
* **Static by default**: the executable is fully statically linked, no runtime dependencies
* **Multiple arch**: `aarch64`, `arm`, `x86_64`, `i686`
* Prebuilt binaries are built by GitHub Actions and published to [Releases](../../releases)

## Download

Download `android-sdk-tools-static-<arch>.zip` from [Releases](../../releases).
The release tag is the `PLATFORM_TOOLS_VERSION` of
[`patches/misc/platform_tools_version.h`](patches/misc/platform_tools_version.h), e.g. `36.0.0`.

## Build

### 1. Dependencies

`git` `patch` `python3` `cmake (>= 3.19)` `ninja` `bison` `flex` `go`, a host C/C++ compiler, and the Android NDK.

```bash
# Debian / Ubuntu
sudo apt install git patch python3 cmake ninja-build bison flex golang-go build-essential
```

### 2. Get the source

```bash
python3 get_source.py                       # tag = platform-tools-<PLATFORM_TOOLS_VERSION>
python3 get_source.py --tags platform-tools-36.0.0   # a specific tag
```

### 3. Build adb

```bash
# NDK (default toolchain, bionic, Android executable), static link
python3 build.py --ndk=/path/to/android-ndk-xxx --arch=aarch64 --target adb

# the NDK path can also come from $ANDROID_NDK_HOME / $ANDROID_NDK_ROOT
export ANDROID_NDK_HOME=/path/to/android-ndk-r29
python3 build.py --arch=aarch64 --target adb

# other archs: arm / x86_64 / i686 (android abi names are accepted too)
python3 build.py --arch=arm --target adb
```

The build directory is `build/<toolchain>-<link>-<arch>`, the executable is `build/<...>/bin/platform-tools/adb`.

### Docker

```bash
docker build --platform linux/amd64 -t android-sdk-tools .
docker run --rm -it --platform linux/amd64 -u "$(id -u):$(id -g)" -v "$PWD:/work" android-sdk-tools \
    sh -c "python3 get_source.py && python3 build.py --arch aarch64 --target adb"
```

## GitHub Actions

[`.github/workflows/build.yml`](.github/workflows/build.yml) builds `adb` for every arch on every push
with the NDK, then publishes the zips to [Releases](../../releases).

## License

[Apache License 2.0](LICENSE.txt)
