# DongShan-edk2

`edk2-stable202308-264` 是本项目早期可编译版本的分支名称。
采用 Sophgo 于 2023-11-02 的 SG2042 配套版本
[`1e7b720`](https://github.com/sophgo/sophgo-edk2/commit/1e7b720306509b1c5e5d1d207277ffb692153ce6)，
已验证 Pisces 双路服务器配置的 RELEASE 固件编译。
当前迁移完成的最新版本保存在 `edk2-stable202610` 分支。

| 子仓库 | 固定提交 |
| --- | --- |
| edk2 | `0355e559c6c4a7d3602120045d74e3a1e792fec6` |
| edk2-platforms | `654e02779068c0a7d272af6f2dc31d22174067cf` |
| edk2-non-osi | `bbc2432146957b9095824e02e95021b4811343f9` |

## 获取源码

先初始化三个核心仓库，再下载 SG2042 编译所需的嵌套依赖：

```sh
git clone --branch edk2-stable202308-264 https://github.com/AII-SDU/DongShan-edk2.git
cd DongShan-edk2
git submodule update --init edk2 edk2-platforms edk2-non-osi
git -C edk2 submodule update --init \
  BaseTools/Source/C/BrotliCompress/brotli \
  MdeModulePkg/Library/BrotliCustomDecompressLib/brotli \
  CryptoPkg/Library/OpensslLib/openssl \
  CryptoPkg/Library/MbedTlsLib/mbedtls \
  MdePkg/Library/BaseFdtLib/libfdt \
  MdePkg/Library/MipiSysTLib/mipisyst \
  MdeModulePkg/Universal/RegularExpressionDxe/oniguruma
```

## 编译固件

已验证的环境为 Ubuntu 22.04、RISC-V GCC 10.2.0、binutils 2.35.1、
主机 GCC 11、Python 3 和 UUID 开发库。
在仓库根目录执行以下命令，使用历史 SG2042_EVB 入口编译 Pisces 配置：

```sh
export WORKSPACE="$PWD"
export PACKAGES_PATH="$WORKSPACE/edk2:$WORKSPACE/edk2-platforms:$WORKSPACE/edk2-non-osi"
export GCC5_RISCV64_PREFIX=riscv64-unknown-elf-
export PYTHON_COMMAND=python3
source edk2/edksetup.sh BaseTools
make -C edk2/BaseTools -j8
build -a RISCV64 -t GCC5 -b RELEASE \
  -p Platform/Sophgo/SG2042_EVB_Board/SG2042.dsc \
  --pcd gSophgoSG2042PlatformPkgTokenSpaceGuid.PcdMangoPcieEnableMask=0x4
```

产物为 `Build/SG2042_EVB/RELEASE_GCC5/FV/SG2042.fd`。
本分支没有 `SRA1-20` 平台目录，不需要额外的 `external-modules`。

首次发布时，先推送三个核心子仓库的 `edk2-stable202308-264` 分支，
再推送主仓库的同名分支，以便下载完整的历史源码组合。
