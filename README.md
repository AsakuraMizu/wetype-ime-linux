# WeType Linux

在 Linux 上通过 Fcitx 5 使用微信输入法（WeType）引擎。引擎是 Android ARM64 版本，由 QEMU user 模式运行在 Android 自己的 C 运行时（AOSP bionic）上。

> **非官方项目**，与腾讯无关，也未获其认可。WeType、微信输入法、微信是腾讯的商标。使用 WeType 引擎须遵守腾讯的相关条款。

本仓库和 AppImage 只包含本项目自己的代码，以及可再分发的运行时（QEMU、取自 AOSP 系统镜像的 bionic），不包含任何 WeType 文件。安装时会从腾讯官方服务器下载 WeType 3.5.4 APK，校验 SHA-256 后在本机打补丁，思路类似 Minecraft 的 Paper。

## 安装

先安装依赖：

```sh
sudo apt install python3 unzip curl          # Debian / Ubuntu
sudo dnf install python3 unzip curl          # Fedora
sudo pacman -S --needed python unzip curl    # Arch
```

然后运行：

```sh
./WeTypeIME-Engine-x86_64.AppImage install            # 安装到 ~/.local；用 sudo 则安装到 /usr
```

安装完成后重启 Fcitx5，并在输入法配置中添加“微信拼音”。

其他命令：

```sh
./WeTypeIME-Engine-x86_64.AppImage install --apk 文件   # 使用已下载的 APK（仅支持 3.5.4，其他版本未经测试，会被拒绝）
./WeTypeIME-Engine-x86_64.AppImage uninstall            # 卸载（保留词库和用户数据）
./WeTypeIME-Engine-x86_64.AppImage demo nihao           # 命令行测试候选词
```

APK 约 214 MB，只下载一次，缓存在 `~/.cache/wetype-ime`。用户学习数据在 `~/.local/share/wetype-ime`。

## 从源码构建

构建环境为 x86_64 的 Debian / Ubuntu：

```sh
sudo apt install build-essential cmake libfcitx5core-dev file qemu-user-static \
  e2fsprogs unzip python3 curl
```

还需要 Android NDK（已用 r27 验证），用 `ANDROID_NDK_HOME` 指向它。构建时会从 Google 下载 AOSP Android 9 的 ARM64 模拟器系统镜像（约 407 MB，缓存在 `.deps/aosp/`），校验 SHA-256 后从中提取 bionic 运行时。

```sh
scripts/e2_img.sh        # 构建插件、harness 并打包 AppImage（不需要 APK）
```

没有 FUSE 时（容器、虚拟机）请设置 `APPIMAGE_EXTRACT_AND_RUN=1`。打包时如果发现任何来自 APK 的文件，会拒绝打包。

在源码树中直接调试引擎：

```sh
scripts/20_build.sh          # 用 NDK 构建 harness 和 libandroid.so 替身，提取 bionic 到 runtime/sysroot
scripts/prepare_assets.sh    # 下载并校验 APK 到 .deps/
scripts/10_patch_libs.sh     # 拷贝 APK 中的引擎库并打补丁，输出到 runtime/
```

插件日志默认写入 `/tmp/wetype-harness.log`。

## 目录结构

- `fcitx5-wetype/`：Fcitx 5 插件
- `harness/`：伪造 JNI 环境、驱动引擎的 ARM64 程序
- `shim/`：`libandroid.so` 替身
- `scripts/`：下载、补丁、构建、打包和测试脚本

## 许可证

本项目使用 GPL-3.0-or-later，见 [LICENSE](LICENSE)；`harness/jni.h` 取自 AOSP，使用 Apache-2.0。AppImage 内附带的 QEMU（GPL-2.0）和 AOSP 组件（bionic、liblog、libc++、zlib）的许可说明见镜像内的 `usr/share/doc/wetype-ime/THIRD-PARTY.md`。WeType 引擎和词库归腾讯所有，不在本许可范围内，本项目也不分发它们。
