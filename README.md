# kluster-vendor-runc

把上游 runc 的二进制重新打包成一个多架构 OCI 镜像，用于离线 / 内网环境分发。

[![Release](https://github.com/cpaas-dev/kluster-vendor-runc/actions/workflows/release.yml/badge.svg)](https://github.com/cpaas-dev/kluster-vendor-runc/actions/workflows/release.yml)
[![Scan upstream](https://github.com/cpaas-dev/kluster-vendor-runc/actions/workflows/scan-upstream.yml/badge.svg)](https://github.com/cpaas-dev/kluster-vendor-runc/actions/workflows/scan-upstream.yml)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
![Platforms](https://img.shields.io/badge/platforms-linux%2Famd64%20%7C%20linux%2Farm64-informational)

```bash
docker pull ghcr.io/cpaas-dev/kluster-vendor-runc:v1.5.3
```

## 这是什么

离线部署容器运行时，需要 GitHub Release 上的 runc 发布件。containerd 2.x 的发布包
不再带 runc，需要单独分发。runc 上游发的是一个静态链接的二进制，本仓库把它按目标架构
放进镜像，只装二进制，不带镜像、不带配置。

## 镜像内容

```plain
/cpaas/vendor/runc/
├── runc                                   # 目标架构
├── runc.sha256sum
└── runc.<arch>.asc                        # 上游 GPG 签名，留作凭证
```

下游校验：

```bash
cd /cpaas/vendor/runc

sha256sum -c runc.sha256sum
```

说明：

- 镜像 tag 和 Release 与上游 tag 1:1，上游本身就带 `v`：`v1.5.3` → `v1.5.3`
- 二进制取自上游 `runc.<arch>`，原样落盘后改名成 `runc`
- 构建期会用上游的 `runc.sha256sum` 校验下载的二进制，不过直接失败。
  镜像里的 `runc.sha256sum` 只是把上游那一行的文件名改成了 `runc`，哈希值和上游一致
- 上游的 `.asc` 一并打进镜像，留作凭证，可以离线自行验签
- 构建期可选开启 GPG 校验，见 [自定义构建](docs/BUILD_YOUR_OWN.md)，默认关闭
- 构建期还会检查落盘文件确实是对应架构的 ELF（magic + `e_machine`）
- 只含目标架构。`linux/arm64` 的镜像里只有 arm64 的 runc

## 文档

- [自定义构建](docs/BUILD_YOUR_OWN.md) —— build-args、下载加速、GPG 校验
- [发布流程](docs/RELEASE.md) —— tag 规则、定时扫描、本地调试脚本
- [贡献指南](docs/CONTRIBUTING.md)

## 相关仓库

- runc <https://github.com/opencontainers/runc>

## 许可

本仓库的构建脚本以 Apache-2.0 发布。打包进镜像的上游产物遵循其原始许可：
runc（Apache-2.0）。上游二进制静态链接了 libseccomp（LGPL-2.1），对应源码见上游
Release 附带的 `libseccomp-<ver>.tar.gz`。
