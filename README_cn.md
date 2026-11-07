# android-sdk-tools（仅 adb）

[English](README.md) | 简体中文

使用 CMake 从 AOSP 上游源码构建 `adb`，产出**安卓可执行文件**
（Android NDK / bionic，默认全静态链接）。

基于 [lzhiyong/android-sdk-tools](https://github.com/lzhiyong/android-sdk-tools)（Apache-2.0），
裁剪为**只构建 adb**：`repository.json` 只拉取 adb 所需的上游 AOSP 仓库（约 25 个，原版 40+ 个），
CMake 工程只定义 `adb` 目标。

* **只构建 adb**：`get_source.py` 只拉取 adb 依赖的源码，`build.py` 只构建 `adb` 目标
* **默认静态链接**：可执行文件完全静态，无运行时依赖
* **多架构**：`aarch64`、`arm`、`x86_64`、`i686`
* 预编译产物由 GitHub Actions 构建并发布到 [Releases](../../releases)

## 下载

从 [Releases](../../releases) 下载 `android-sdk-tools-static-<arch>.zip`。
Release 标签即 [`patches/misc/platform_tools_version.h`](patches/misc/platform_tools_version.h)
中的 `PLATFORM_TOOLS_VERSION`，例如 `36.0.0`。

## 构建

### 1. 依赖

`git` `patch` `python3` `cmake (>= 3.19)` `ninja` `bison` `flex` `go`、本机 C/C++ 编译器、Android NDK。

```bash
# Debian / Ubuntu
sudo apt install git patch python3 cmake ninja-build bison flex golang-go build-essential
```

### 2. 拉取上游源码

```bash
python3 get_source.py                                # 标签 = platform-tools-<PLATFORM_TOOLS_VERSION>
python3 get_source.py --tags platform-tools-36.0.0   # 指定标签
```

### 3. 构建 adb

```bash
# NDK 工具链（默认，bionic，安卓可执行文件），静态链接
python3 build.py --ndk=/path/to/android-ndk-xxx --arch=aarch64 --target adb

# NDK 路径也可通过环境变量提供
export ANDROID_NDK_HOME=/path/to/android-ndk-r29
python3 build.py --arch=aarch64 --target adb

# 其他架构：arm / x86_64 / i686（也接受 android abi 名称）
python3 build.py --arch=arm --target adb
```

构建目录为 `build/<toolchain>-<link>-<arch>`，产物位于 `build/<...>/bin/platform-tools/adb`。

### Docker

```bash
docker build --platform linux/amd64 -t android-sdk-tools .
docker run --rm -it --platform linux/amd64 -u "$(id -u):$(id -g)" -v "$PWD:/work" android-sdk-tools \
    sh -c "python3 get_source.py && python3 build.py --arch aarch64 --target adb"
```

## GitHub Actions

[`.github/workflows/build.yml`](.github/workflows/build.yml) 在每次 push 时用 NDK 为各架构构建 `adb`，
并把压缩包发布到 [Releases](../../releases)。

## 许可证

[Apache License 2.0](LICENSE.txt)
