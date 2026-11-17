## 打包

```bash
# 构建多架构镜像 amd64/arm64
docker buildx build --platform linux/amd64,linux/arm64 \
  -t xxx .
```

## 指定版本

```bash
# 和上游 tag 一致，带 v 前缀
docker buildx build \
  --build-arg INST_RUNC_V=v1.5.3 \
  -t ...
```

## 下载加速

```bash
# 国内访问加速
docker buildx build \
  --build-arg GITHUB_URL=https://gh-proxy.org/https://github.com \
  -t ...

# 本地自建 Nexus 代理
docker buildx build \
  --build-arg GITHUB_URL=https://nexus-mirror.alpha-quant.tech/repository/github \
  -t ...
```

## GPG 校验（可选）

上游的 `runc.sha256sum` 是 GPG clearsign 过的，每个二进制也有 detached 签名
`runc.<arch>.asc`。默认只用 `runc.sha256sum` 里的哈希校验，不验签。
想在构建期验签，填入签名者指纹即可：

```bash
docker buildx build \
  --build-arg RUNC_GPG_KEYS="<签名者指纹>" \
  -t ...
```

多个指纹用空格分隔。开启后构建会 `dnf install gnupg2` 并从 keyserver 拉公钥，
`runc.sha256sum` 和每个二进制的签名都要验过，不过直接失败。keyserver 可以换：

```bash
--build-arg GPG_KEYSERVER=hkps://keyserver.ubuntu.com
```

上游仓库根目录的 `runc.keyring` 列出了维护者公钥，指纹自己从那里核对，本仓库不硬编码——
写死一个没核对过的指纹，比不校验更糟。

## 其他 build-args

```bash
# 基础镜像
BASE_IMAGE=docker.io/rockylinux/rockylinux:10.2.20260525.0

# 下载哪些架构的二进制，空格分隔。
# 最终镜像只装 TARGETARCH，这里改小只能省下载量，改小于目标架构集合会构建失败。
# 上游还发 386 / armel / armhf / ppc64le / riscv64 / s390x
VENDOR_ARCHES="amd64 arm64"
```
