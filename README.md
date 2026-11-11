# DongShan-edk2

`edk2-stable202611` 分支迁移自 [Sophgo 配套源码](https://github.com/sophgo/sophgo-edk2/tree/c61f6d985148b96a40fc0708d8f97a33f6d4785c)
（`devel-sg2042` 主仓库版本 `c61f6d9`），包含 SG2042 v1.5.1 平台固件，
并保留配套源码中的 SG2044 平台支持。该名称是 DongShan 的迁移分支名。
`edk2`、`edk2-platforms`、`edk2-non-osi` 通过 Git 子模块固定到配套版本；
`external-modules` 包含跨架构 UEFI 模拟功能所需的依赖。
SG2042 已默认编入模拟器，通过固件设置中的运行时开关控制。
来源版本、迁移范围和发布顺序见 [迁移记录](docs/sophgo-migration.md)。

## 获取源码

```sh
git clone --branch edk2-stable202611 --recurse-submodules https://github.com/AII-SDU/DongShan-edk2.git
cd DongShan-edk2
```

已有仓库更新后执行：

```sh
git submodule update --init --recursive
```

普通 `git clone` 和 GitHub ZIP 下载不会自动包含子模块内容。

## 编译固件

准备 RISC-V GCC 工具链（本分支已验证 GCC 15.2.0）、主机 GCC、Make、Python 3
和 UUID 开发库。启用 ACPI 时需要 iASL 20241212 或兼容版本。

在仓库根目录编译 SRA1-20 RELEASE 固件：

```sh
export WORKSPACE="$PWD"
export PACKAGES_PATH="$WORKSPACE/edk2:$WORKSPACE/edk2-platforms:$WORKSPACE/edk2-non-osi:$WORKSPACE/external-modules"
export GCC5_RISCV64_PREFIX=riscv64-unknown-elf-
export PYTHON_COMMAND=python3
source edk2/edksetup.sh BaseTools
make -C edk2/BaseTools -j8
build -a RISCV64 -t GCC5 -b RELEASE -p Platform/Sophgo/SG2042Pkg/SRA1-20/SRA1-20.dsc
```

产物为 `Build/SRA1-20/RELEASE_GCC5/FV/SRA1-20.fd`。
使用 `-b DEBUG` 编译调试版本，添加 `-D ACPI_ENABLE` 启用 SG2042 ACPI。
SG2042-EVB、MilkV-Pioneer 的平台配置也位于 `Platform/Sophgo/SG2042Pkg/`；
SG2044 使用 `Platform/Sophgo/SG2044Pkg/SG2044.dsc`，默认启用 ACPI。

## SG2042 固件打包

`tools/` 保留 Sophgo 的独立构建、打包及签名工具；`keys/` 保留其公开的
默认示例密钥，`release-note/` 保留 SG2042 发布记录。

以下命令编译带 ACPI 的 SRA1-20 RELEASE 固件，并打包 `firmware.bin`：

```sh
export PLAT=SRA1-20
export RISCV64_ELF_CROSS_COMPILE=riscv64-unknown-elf-
source tools/envsetup.sh
set -eo pipefail
build_rv_edk2
build_rv_firmware_bin
```

产物位于 `Build/SRA1-20/firmware/`。该入口还支持 `SG2042-EVB`、
`MilkV-Pioneer`。`build_rv_firmware_image` 可进一步生成 `firmware.img`，
需要 `parted`、`kpartx`、`dosfstools` 及 sudo 权限。

首次发布时，先将三个核心子仓库的 `edk2-stable202611` 分支推送至各自的
`AII-SDU` 远端，再推送主仓库，确保递归克隆可以下载固定的子模块版本。
