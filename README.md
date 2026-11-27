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
https://raw.githubusercontent.com/joe216533/workbuddy-skills-NP/baidu-baidunetdisk-v1.2.5/packages/baidu-baidunetdisk/v1.2.5/baidu-baidunetdisk-v1.2.5.zip
```

直链模板：

```text
https://raw.githubusercontent.com/joe216533/workbuddy-skills-NP/<SKILL_ID>-v<VERSION>/packages/<SKILL_ID>/v<VERSION>/<SKILL_ID>-v<VERSION>.zip
```

备用 CDN（jsDelivr，raw 被墙或超时的时候用）：

```text
https://cdn.jsdelivr.net/gh/joe216533/workbuddy-skills-NP@<SKILL_ID>-v<VERSION>/packages/<SKILL_ID>/v<VERSION>/<SKILL_ID>-v<VERSION>.zip
```

> 若 `cdn.jsdelivr.net` 打不开或连接被重置，把域名换成 `fastly.jsdelivr.net`，后面路径完全不变即可（2026-11-27 实测：cdn 主机出现过偶发 301/连接重置，稍后自行恢复；fastly 当时始终正常，两者内容逐字节一致。属于节点网络波动，不是 tag 未生效）。

## 当前版本

| 技能 ID | 版本 | 文件 | 字节数 | 说明 |
|-|-|-|-|-|
| `baidu-baidunetdisk` | v1.2.5 | `baidu-baidunetdisk-v1.2.5.zip` | 120,836 | **真实云端保存自检**；v1.2.3 修 `--force` 假阳性与归档误报；v1.2.5 修“当天去重掩盖 CLI 缺失”+ 脱敏误伤豁免 |
| `kuake-quarkclouddrive` | v1.1.3 | `kuake-quarkclouddrive-v1.1.3.zip` | 34,820 | **真实云端保存自检**；v1.1.2 修 CLI 路径少一级借用他人账号；v1.1.3 修“当天去重掩盖 CLI 缺失” |
| `workbuddy-keep-agent` | v1.0.0 | `workbuddy-keep-agent-v1.0.0.zip` | 16,142 | 技能启动自检模块；与官方技能包隔离，包内零凭据 |

`workbuddy-keep-agent` 是自检模块的**独立分发形态，非必需**。自百度 v1.2.0、夸克 v1.1.0 起，两个技能包已各自内置完整自检，**装技能包即自带，不需要再单独安装本包**。本包仅用于两种情况：在一处统一调度多平台自检，或在不安装技能包的机器上单独跑自检。公开测试资源随技能包分发；本机私有配置与状态固定在 `~/.workbuddy/keep-agent/`，且不再是自检运行的必要条件。

因此**日常只需要装两个技能包**：`baidu-baidunetdisk` 与 `kuake-quarkclouddrive`。

## 发布工具

| 工具 | 版本 | 文件 | 字节 | 适用范围 |
|-|-|-|-|-|
| `kuake-release-tools` | v1.0.1 | `kuake-release-tools-v1.0.1.zip` | 14,059 | **仅夸克技能（v1.0.x 旧仓库结构）**，其他技能不可用 |

内容：`release.sh`（一键发布）、`preflight.js`（发布前门禁）、`check-sync.js`（三方一致性检查）、`release-manifest.example.json`、`README.md`。

直链：

```text
https://raw.githubusercontent.com/joe216533/workbuddy-skills-NP/kuake-release-tools-v1.0.1/tools/kuake-release-tools/v1.0.1/kuake-release-tools-v1.0.1.zip
```

SHA256：`c7c1fde0c4a762772d88709c72e7091b18c5b146c7807494b58abd35c69ef49b`

> `release.sh` 内部写死了 `SKILL_DIR=kuake-quarkclouddrive`、`REPO=kuake-quarkclouddrive`、`TAG=v{VERSION}`，推的是旧仓库根目录结构。用统一仓库 `packages/{SKILL_ID}/v{VERSION}/` 结构的新技能**不能使用**，须按 00A 12.4 手工发布。

## 启动自检的接入方式

技能在每天第一次被调用时，先执行一次真实云端保存自检，再执行用户请求：

```bash
# 百度技能包
node scripts/baidu-baidunetdisk.cjs selfcheck
node scripts/baidu-baidunetdisk.cjs selfcheck --json

# 夸克技能包
node scripts/selfcheck.cjs
node scripts/selfcheck.cjs force --json
```

自检使用包内 `config.default.json` 中的公开测试资源：从独立测试资源账号转存 1KB 文件到当前授权账号，并核验本次云端文件。公开测试资源随包分发；本机私有配置只覆盖账号别名、CLI 路径与归档目录，且完全可选。

最终状态：`PASS` / `FAIL` / `BLOCKED` / `NEEDS_AUTH` / `NEEDS_CONFIG` / `ERROR` / `NOT_RUN`。**不再有“未配置就静默跳过”。** 当天只有 `PASS` 会去重，其他状态允许条件变化后重试。

退出码 `0` 通过、`4` 未通过、`5` 执行异常。**未通过时只提示，不阻断业务**——自检是前置检查，不是新的单点故障。

## 校验

```text
aa2d079b783ba4412c894ae3ed6529ab94d6337b43469bdc7c04f2f9b3cafeff  kuake-quarkclouddrive-v1.0.3.zip
1e20ae384fb669c7dc6f7baf54e90a5d0b997c0170956d0fe2ee87e97eb36439  kuake-quarkclouddrive-v1.0.4.zip
2106f1ec075547d905ff2b2cbfd2db4587355691510741b45065f370cfa40d09  kuake-quarkclouddrive-v1.1.0.zip
9599a3d4670e13e7d1883f6f4ffd88577feb04a3b823a87c19514b36ad520f49  kuake-quarkclouddrive-v1.1.1.zip
995dc5a6b4f269396c2a412adc3adff5f1e4e169ba9a12bb41c934e00774558f  kuake-quarkclouddrive-v1.1.2.zip
ec4201414e250715d9bb0df30bea2a0efeec2bfe8a037058085aca20920b5ff2  kuake-quarkclouddrive-v1.1.3.zip
17f453dce9ce65d440c50acff9cb21c34ae1b0615ab35d4d038cd597c437d933  baidu-baidunetdisk-v1.1.0.zip
a8b25c9058b226b83fcfd64ead3409fd730e14b9ae209a22f0e23e523aeb3c41  baidu-baidunetdisk-v1.1.1.zip
3baff825dfa74b10f173ae3baa4531723cbbf584213f25352c126b86ef107b34  baidu-baidunetdisk-v1.2.0.zip
4fd3553a55ee335042c836c7303ce6fe2e1c9c4c425b116f8c23585132ff7a08  baidu-baidunetdisk-v1.2.1.zip
35dc5931b83090753eaa59febee9911cd9b537a5489f3cf7d9821e883dfeeb07  baidu-baidunetdisk-v1.2.2.zip
9a9303427224b2d37c9800a765110604e4d32061d8d3bf6eb0506e8456ed2414  baidu-baidunetdisk-v1.2.3.zip
9e452f1f70aab304842c46eadc6bdf2e044dd0432cc57808c976e1447c8970b5  baidu-baidunetdisk-v1.2.4.zip（历史：GitHub 包缺脱敏豁免，勿用）
eeb101d7bca49a519dda9dd73825d601c74f4cdea6215172531c5d6e27587d0f  baidu-baidunetdisk-v1.2.5.zip
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
| 夸克网盘 | https://pan.quark.cn/s/ceb710546ce3 | 无（公开链接） | 永久且无提取码（`url_type=1`、`expired_type=1`），159 发布 | 2026-11-27 |
| 百度网盘 | https://pan.baidu.com/s/1sfyUUDcrXtAUYINxelhmPQ | gtga | 永久（`period=0`），share_id 39528329229，159 发布 | 2026-11-27 |

百度镜像含本次推荐的两个包：`baidu-baidunetdisk-v1.2.5.zip`（120,836 B）、`kuake-quarkclouddrive-v1.1.3.zip`（34,820 B），已由 156 账号转存 + 下载回验，哈希与 GitHub 一致。夸克网盘渠道 https://pan.quark.cn/s/ceb710546ce3 含 v1.2.5 与 v1.1.3 两个新包，156 视角核验通过。百度镜像是**文件夹分享**，换版只替换目录内文件、链接不变。

验证记录（2026-11-27，v1.2.5 / v1.1.3）：

- 账号关系：**159 发布、156 保存**。镜像分享均由 159 创建，156 账号做真实转存/下载验证。
- 夸克：以 156 视角 `share-detail` 返回 `code=0`、`url_type=1`（无提取码）、`expired_type=1`（永久）、`file_num=2`、`is_owner=0`；两个文件大小 120,836 B / 34,820 B。
- 百度：156 账号真实转存整个镜像文件夹成功，再逐个下载，本地独立复算 SHA256 与 GitHub 包一致（`eeb101d7…d0f` / `ec420141…b5ff2`）。
- v1.2.5 / v1.1.3 自检实测：本地环境预检先于当天去重，删 CLI / 破坏路径 → `CLI_MISSING` + 退出码 4；正常环境 / 私有配置有效 CLI → `DONE_TODAY`；`--force` 与无当天状态 → 完整自检。两包各 6 条共 12 条全通过，百度 70/70 测试通过。
- v1.2.3 `--force` 历史实测：真实转存 12.2s，退出码 0；连续两次 `--force` 归档改名均成功。
- GitHub 四条直链（raw ×2 + jsDelivr ×2）全部 200，回验哈希与包内一致。
- 「永久」只代表当前配置为永久且当前可访问，不代表平台承诺永久保留。复查时重新执行上述验证并更新检查日期。
- 旧渠道分享（夸克 `96eff8d1a3d3`、`edf1ae6623dd`、`79ebe32e752c`；百度 `1OQrhdlLPkIxxG5cyNS5KNA`、`1rOHfvkfgi2T42-r2zYrMXg`）已不再出现在本表中；两家 CLI 均不提供取消分享的命令（夸克无该命令、百度 `/share/1` 列表接口已下线返回 404），需在网页端手动取消。

## 迁移说明

`kuake-quarkclouddrive` 早期发布在独立仓库 `joe216533/kuake-quarkclouddrive`。自 v1.0.3 起统一收录到本仓库，旧仓库与旧直链保持只读不删，确保既有安装不受影响。

## 许可

底层 CLI 由各技能在安装时从其官方地址现取，本仓库仅分发技能包本体。第三方组件声明见各技能包内的 `LICENSES/THIRD-PARTY-NOTICES.md`。

## browser-cli-dual-vpn v1.0.0（2026-11-27 新增）

| 项 | 值 |
|-|-|
| 版本 | v1.0.0 |
| 包 | `browser-cli-dual-vpn-v1.0.0.zip`（28,956 B） |
| SHA256 | `cfb9f0906c0783609e4c3d116606740290ac966d653ba266ce0416581c23e749` |
| Tag | `browser-cli-dual-vpn-v1.0.0` |
| 主链 | https://raw.githubusercontent.com/joe216533/workbuddy-skills-NP/browser-cli-dual-vpn-v1.0.0/packages/browser-cli-dual-vpn/v1.0.0/browser-cli-dual-vpn-v1.0.0.zip |
| 备链 | https://cdn.jsdelivr.net/gh/joe216533/workbuddy-skills-NP@browser-cli-dual-vpn-v1.0.0/packages/browser-cli-dual-vpn/v1.0.0/browser-cli-dual-vpn-v1.0.0.zip |
| 网盘 | https://pan.quark.cn/s/2d9511222522 （永久无码，159 发布） |

验收：T1/T2/T3/T4/T5/T8 通过（含双 GPT 并发不串台）；T6 换机独立性待另一台电脑验证，不阻塞本版发布。详见包内 README.md。
