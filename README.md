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
| `baidu-baidunetdisk` | v1.1.0 | `baidu-baidunetdisk-v1.1.0.zip` | 98,124 | 新增 `rename` 重命名；转存/上传/分享/删除/配额/校验全量能力 |
| `kuake-quarkclouddrive` | v1.0.3 | `kuake-quarkclouddrive-v1.0.3.zip` | 14,376 | 上传/下载/分享/转存/批量重命名；本仓库为迁移收录，内容与旧仓库逐字节一致 |

## 校验

```text
aa2d079b783ba4412c894ae3ed6529ab94d6337b43469bdc7c04f2f9b3cafeff  kuake-quarkclouddrive-v1.0.3.zip
17f453dce9ce65d440c50acff9cb21c34ae1b0615ab35d4d038cd597c437d933  baidu-baidunetdisk-v1.1.0.zip
```

安装方必须下载后核对 SHA256，与本表或同目录 `.sha256` 文件一致才可安装。

## 网络要求

- `baidu-baidunetdisk`：无特殊要求。
- `kuake-quarkclouddrive`：夸克对境外出口有风控，走代理会报 `1007` 或 TLS 断开。**执行夸克操作前必须切国内直连**；仅下载本仓库分发包时可走代理。

## 迁移说明

`kuake-quarkclouddrive` 早期发布在独立仓库 `joe216533/kuake-quarkclouddrive`。自 v1.0.3 起统一收录到本仓库，旧仓库与旧直链保持只读不删，确保既有安装不受影响。

## 许可

底层 CLI 由各技能在安装时从其官方地址现取，本仓库仅分发技能包本体。第三方组件声明见各技能包内的 `LICENSES/THIRD-PARTY-NOTICES.md`。
