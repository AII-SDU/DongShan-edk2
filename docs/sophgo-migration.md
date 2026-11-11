# edk2-stable202611 的 Sophgo 固件迁移

本分支从 DongShan-edk2 的 `dongshan-edk2` 提交 `701b474` 建立，
迁移本地 sophgo-edk2 的 `devel-sg2042` 配套代码，主仓库提交为
`c61f6d985148b96a40fc0708d8f97a33f6d4785c`（2026-10-27）。
`edk2-stable202611` 是本项目的分支名称，未改用同名 TianoCore 上游基线。

## 固定版本

| 子模块 | 迁移前 | 迁移后 |
| --- | --- | --- |
| edk2 | `e1d3e494c275068932730e3e6bf44139d3675dae` | `42af26f57a8f109de3528e1a57d8f20d01562712` |
| edk2-platforms | `02e6b7618e4090cad54b2724f6a535990658a539` | `5cab5f35f3ffb325f02680933c413fa33f88a1d6` |
| edk2-non-osi | `3c7fbde3a7e33710a8d62267452a5acfde3c315d` | `d1ab71b196d55d3b8773bb7192723f0c952e674b` |
| external-modules/MultiArchUefiPkg | `93f4a2f3399c9f55cbe07f30905aa9c28e383873` | `94e536283a6e23af188f7cd78f417682dde316fe` |
| external-modules/unicorn | `a105afc5170b5608dfdf8a3cc6cdd517ed42e4c4` | `9a1dfb970ed091c12c20ab4eb9c6bacd7ca869c1` |

五个子模块均在上述配套提交建立本地 `edk2-stable202611` 分支。
迁移前的版本均为迁移后版本的祖先，无 DongShan 独有提交需要合并。
核心子模块继续使用 AII-SDU 远端，外部依赖继续使用 Sophgo 远端。
普通递归克隆按主仓库的 gitlink 固定版本；`.gitmodules` 中的 `branch`
仅影响 `git submodule update --remote`。

## 代码范围

- `edk2-platforms/Platform/Sophgo`：SG2042-EVB、MilkV-Pioneer、SRA1-20
  和 SG2044 的 DSC/FDF、平台配置及文档。
- `edk2-platforms/Silicon/Sophgo`：芯片驱动、库、ACPI、DebugConfigDxe
  及 SG2042 v1.5.1 的 PCIe DMA 范围修复。SG2042 串口默认关闭，
  串口和跨架构模拟器使用运行时配置开关。
- `edk2`：配套的内存分配及镜像验证更新。
- `edk2-non-osi`：预编译 OpenSBI、ZSBL、FIP、设备树及 GOP 等组件。
- `external-modules`：配套模拟器和 Unicorn，包含新版本 GCC 的编译修复。
- `tools/`、`keys/`、`release-note/`：从 Sophgo 主仓库迁入构建、
  打包、签名工具、公开默认示例密钥和发布说明。工具仅清理了行尾空白。
- `.github/workflows/build.yml`：迁入 SG2042 三个平台的固件构建流水线，
  将 checkout 改为当前 DongShan 仓库，产物名称相应修改，并固定安装
  平台 ACPI 表所需的 iASL 20241212。

各子仓库历史和许可证保持原样，构建产物未纳入版本控制。

## 本地验证

2026-11-11 使用 xPack RISC-V GCC 15.2.0、`GCC5` 工具链标签和
iASL 20241212 验证。xPack 的编译入口通过本地包装脚本将默认架构设为
`rv64gc/lp64d`，与 Sophgo 所用的 riscv64 工具链默认配置一致。

| 平台 | 配置 | 结果 |
| --- | --- | --- |
| SRA1-20 | RISCV64 RELEASE，启用 ACPI | 通过 |
| SG2042-EVB | RISCV64 RELEASE，启用 ACPI | 通过 |
| MilkV-Pioneer | RISCV64 RELEASE，启用 ACPI | 通过 |
| SG2044 | RISCV64 RELEASE，启用 ACPI | 通过 |

BaseTools 重新编译成功，内置 303 项测试全部通过。
SRA1-20 的 `firmware.bin` 打包成功，版本为 `1.5.1`，14 个分区均与
源组件逐字节一致，分区无重叠，提取出的 FD 与编译产物一致。
示例密钥的签名、验签和公钥 DER 转换通过；Shell 语法及 CI YAML 检查通过。

本机产物存放于 `Build/edk2-stable202611/`，包含四个平台的 FD、
SRA1-20 的 `firmware.bin`、`SHA256SUMS` 和来源版本记录。
构建脚本、报告及日志位于 `.local/edk2-stable202611-validation/`，
使用独立工作目录生成产物，保留原有 `Build/` 下的固件。
本次未执行 `firmware.img` 的特权磁盘映像制作，也未进行上板验证。

## 发布顺序

本次迁移在本地完成。发布到远端时先发布核心子模块，再发布主仓库：

```sh
git -C edk2 push -u origin edk2-stable202611
git -C edk2-platforms push -u origin edk2-stable202611
git -C edk2-non-osi push -u origin edk2-stable202611
git push -u origin edk2-stable202611
```

两个外部子模块固定为 Sophgo 已有提交，无需发布其本地同名分支。
