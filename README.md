# workbuddy-skills-NP

WorkBuddy 技能统一分发包仓库。所有技能的分发包（zip）都放在这里，作为**唯一分发源**与直链安装入口。

## 目录规范

```
packages/
  <SKILL_ID>/
    v<VERSION>/
      <SKILL_ID>-v<VERSION>.zip          分发包（安装用）
      <SKILL_ID>-v<VERSION>.zip.sha256   校验文件
README.md
```

Tag 规范：`<SKILL_ID>-v<VERSION>`，例如 `baidu-baidunetdisk-v1.1.0`。

**Tag 一旦发布不再改动。** 内容需要变更时递增版本号新建 tag，旧 tag 保留，避免已发出的直链失效。

## 直链安装

把对应技能的直链发给智能体即可，例如：

```text
请根据这个直链下载并安装 baidu-baidunetdisk 技能，并自动完成环境检查、依赖准备和安装验证：
https://raw.githubusercontent.com/joe216533/workbuddy-skills-NP/baidu-baidunetdisk-v1.1.0/packages/baidu-baidunetdisk/v1.1.0/baidu-baidunetdisk-v1.1.0.zip
```

直链模板：

```text
https://raw.githubusercontent.com/joe216533/workbuddy-skills-NP/<SKILL_ID>-v<VERSION>/packages/<SKILL_ID>/v<VERSION>/<SKILL_ID>-v<VERSION>.zip
```

备用 CDN（jsDelivr，raw 被墙或超时的时候用）：

```text
https://cdn.jsdelivr.net/gh/joe216533/workbuddy-skills-NP@<SKILL_ID>-v<VERSION>/packages/<SKILL_ID>/v<VERSION>/<SKILL_ID>-v<VERSION>.zip
```

## 当前版本

| 技能 ID | 版本 | 文件 | 字节数 | 说明 |
|-|-|-|-|-|
| `baidu-baidunetdisk` | v1.1.1 | `baidu-baidunetdisk-v1.1.1.zip` | 115,385 | **内置启动自检**（每天首次调用自动真实转存）；含 rename；上传/下载/分享/转存/删除/配额/校验全量能力 |
| `kuake-quarkclouddrive` | v1.0.4 | `kuake-quarkclouddrive-v1.0.4.zip` | 31,872 | **内置启动自检**（每天首次调用前执行真实转存）；上传/下载/分享/转存/批量重命名 |
| `workbuddy-keep-agent` | v1.0.0 | `workbuddy-keep-agent-v1.0.0.zip` | 16,142 | 技能启动自检模块；与官方技能包隔离，包内零凭据 |

`workbuddy-keep-agent` 是自检模块的**独立分发形态，非必需**。自技能包 v1.1.1 起，百度与夸克两个技能包已各自内置自检，**装技能包即自带，不需要再单独安装本包**。本包仅用于两种情况：在一处统一调度多平台自检，或在不安装技能包的机器上单独跑自检。配置与状态固定在 `~/.workbuddy/keep-agent/`，三个包共用同一份配置，重装任何一方都不丢历史。

因此**日常只需要装两个技能包**：`baidu-baidunetdisk` 与 `kuake-quarkclouddrive`。

## 启动自检的接入方式

技能在每天第一次被调用时，先执行一次自检，再执行用户请求：

```bash
node <keep-agent安装目录>/keep.cjs run --platform quark
node <keep-agent安装目录>/keep.cjs run --platform baidu
```

退出码 `0` 通过、`4` 未通过、`5` 执行异常。**未通过时只提示，不阻断业务**——自检是前置检查，不是新的单点故障。

首次使用需生成配置：

```bash
mkdir -p ~/.workbuddy/keep-agent
cp config.example.json ~/.workbuddy/keep-agent/config.json
node keep.cjs init
```

包内只有 `config.example.json` 模板，不含分享链接、提取码与账号令牌。

## 校验

```text
aa2d079b783ba4412c894ae3ed6529ab94d6337b43469bdc7c04f2f9b3cafeff  kuake-quarkclouddrive-v1.0.3.zip
1e20ae384fb669c7dc6f7baf54e90a5d0b997c0170956d0fe2ee87e97eb36439  kuake-quarkclouddrive-v1.0.4.zip
17f453dce9ce65d440c50acff9cb21c34ae1b0615ab35d4d038cd597c437d933  baidu-baidunetdisk-v1.1.0.zip
a8b25c9058b226b83fcfd64ead3409fd730e14b9ae209a22f0e23e523aeb3c41  baidu-baidunetdisk-v1.1.1.zip
08c68e12a5196ec01767df7b81717a2e2a97fc6ac8687fd8fb759c7776504f13  workbuddy-keep-agent-v1.0.0.zip
```

安装方必须下载后核对 SHA256，与本表或同目录 `.sha256` 文件一致才可安装。

## 网络要求

- `baidu-baidunetdisk`：无特殊要求。
- `kuake-quarkclouddrive`：夸克对境外出口有风控，走代理会报 `1007` 或 TLS 断开。**执行夸克操作前必须切国内直连**；仅下载本仓库分发包时可走代理。

## 网盘镜像

GitHub 直链为主渠道，网盘镜像为备用渠道（GitHub 不可达时使用）。

| 平台 | 链接 | 提取码 | 有效期配置 | 检查日期 |
|-|-|-|-|-|
| 夸克网盘 | https://pan.quark.cn/s/28b6368ad71a | 5gPn | 永久（`expired_type=1`） | 2026-11-27 |
| 百度网盘 | https://pan.baidu.com/s/17iudwvb6pGWCJjZPYEnEpA | uuys | 永久（`period=0`） | 2026-11-26 |

两个镜像目录均含上述三个包的同一版本，与 GitHub 上的文件逐字节一致。

验证记录（2026-11-26）：

- 夸克：`share-detail` 返回 `code=0`、`file_num=3`、`expired_type=1`、`expired_at=4102416000000`；首文件 `kuake-quarkclouddrive-v1.0.3.zip` 14,376 B。
- 百度：156 账号真实转存成功，转存后三个文件大小与源包一致（98,124 / 14,376 / 16,142），验证后已清理。
- 「永久」只代表当前配置为永久且当前可访问，不代表平台承诺永久保留。复查时重新执行上述两项验证并更新检查日期。

## 迁移说明

`kuake-quarkclouddrive` 早期发布在独立仓库 `joe216533/kuake-quarkclouddrive`。自 v1.0.3 起统一收录到本仓库，旧仓库与旧直链保持只读不删，确保既有安装不受影响。

## 许可

底层 CLI 由各技能在安装时从其官方地址现取，本仓库仅分发技能包本体。第三方组件声明见各技能包内的 `LICENSES/THIRD-PARTY-NOTICES.md`。
