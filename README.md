# Sync binaries of golang

从 Golang 官网 <https://go.dev/dl/> 同步 Go 二进制程序(各平台归档包、安装程序及源码包),并通过 GitHub Actions 自动发布到本仓库的 Releases.

## 工作原理

同步流程由 `.github/workflows/dosync.yaml` 定义,核心步骤如下:

1. **解析版本并生成下载清单**
   - 一次性拉取官网全量版本清单 `https://go.dev/dl/?mode=json&include=all`(清单按版本号降序),未指定版本时取最新稳定版,指定版本时按 `version` 精确匹配,且只认 `stable=true` 的条目.
   - 若解析到预发布版本(`rc`/`beta`/`alpha`)则直接报错退出,避免误发非稳定版.
   - 下载清单取自清单中每个文件的 `sha256` 字段,格式为 `<sha256>  <文件名>`,下载 URL 由 `<下载源>/<文件名>` 直接组装,无需写死平台列表,天然适配各版本架构.
2. **并行下载全部二进制包**:基于 `xargs -P` 并发下载(默认 8 路并发,失败自动重试).
3. **校验 SHA256**:用清单中的校验值通过 `sha256sum -c` 逐文件校验,保证文件完整性.
4. **发布到 Releases**:以版本号为 tag(如 `v1.25.0`),上传所有下载文件.

## 触发方式

- **定时触发**:每周四凌晨 4:30(`30 4 * * 4`)自动执行,使用最新稳定版.
- **手动触发**(`workflow_dispatch`):可在 Actions 页面手动运行,支持以下输入参数:

| 参数 | 说明 | 必填 | 示例 |
| --- | --- | --- | --- |
| `binvern` | 指定版本号(如 `1.25.0` 或 `go1.25.0`),不填则使用最新稳定版 | 否 | `1.25.0` |

## 产物

每次同步会在 Releases 中生成一个以版本号命名的发行(如 `Golang v1.25.0 Binaries`),包含对应平台的全部文件(`go1.25.0.linux-amd64.tar.gz`、`go1.25.0.windows-amd64.msi`、`go1.25.0.src.tar.gz` 等),可直接下载使用.

## 目录结构

```
golang/
├── .github/workflows/
│   └── dosync.yaml   # 同步工作流定义
├── LICENSE
└── README.md
```
