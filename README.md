# DongShan-edk2

`dongshan-edk2` 分支采用 [Sophgo 配套源码](https://github.com/sophgo/sophgo-edk2/tree/devel-sg2042)
（主仓库版本 `cd2d36f`），支持 SG2042 / SG2044 固件编译。
`edk2`、`edk2-platforms`、`edk2-non-osi` 通过 Git 子模块固定到配套版本；
`external-modules` 包含可选跨架构 UEFI 模拟功能所需的依赖。

## 获取源码

```sh
git clone --branch dongshan-edk2 --recurse-submodules https://github.com/AII-SDU/DongShan-edk2.git
cd DongShan-edk2
```

已有仓库更新后执行：

```sh
git submodule update --init --recursive
```

普通 `git clone` 和 GitHub ZIP 下载不会自动包含子模块内容。

## 编译固件

准备 RISC-V GCC 工具链（已验证 GCC 13.2.0）、主机 GCC、Make、Python 3
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

首次发布时，先将三个核心子仓库的 `dongshan-edk2` 分支推送至各自的
`AII-SDU` 远端，再提交并推送主仓库，确保他人可以下载固定的子模块版本。
