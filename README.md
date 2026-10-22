# DongShan-edk2

本仓库通过 Git 子模块关联以下三个仓库，并固定各子模块的提交版本：

- [edk2](https://github.com/AII-SDU/edk2)
- [edk2-platforms](https://github.com/AII-SDU/edk2-platforms)
- [edk2-non-osi](https://github.com/AII-SDU/edk2-non-osi)

克隆时使用 `--recurse-submodules`，即可同时下载这三个仓库及其嵌套子模块：

```sh
git clone --recurse-submodules https://github.com/AII-SDU/DongShan-edk2.git
```

如果已经克隆了主仓库，在仓库目录中执行：

```sh
git submodule update --init --recursive
```

普通 `git clone` 和 GitHub 的 ZIP 下载不会自动包含子模块内容。
主仓库更新后，执行以下命令将子模块同步到主仓库记录的版本：

```sh
git pull
git submodule update --init --recursive
```
