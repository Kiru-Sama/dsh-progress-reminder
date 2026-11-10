# 安装

## 方式 A：包名（推荐）

DSH → 设置 → 插件 → 添加插件，输入：

```text
dsh-progress-reminder
```

装完 `文件 → 退出`，再重新启动 DeepSeek Harness。

这个包在 npm 上是公开的：<https://www.npmjs.com/package/dsh-progress-reminder>

> ⚠️ **别装错**：npm 上还有 `dsh-progress-bar`（作者 nikolai1976，纯视觉进度条）
> 和 `dsh-progress-nudge`（作者 hlysx，同样是进度提醒但实现不同）。本插件只叫
> `dsh-progress-reminder`。

包名走不通时，往下看「方式 B / 方式 C」。

## 方式 B：Release tgz

不需要先装任何东西；把固定地址粘进“添加插件”输入框：

```text
https://github.com/Kiru-Sama/dsh-progress-reminder/releases/latest/download/dsh-progress-reminder.tgz
```

钉死版本就用带版本号的地址：

```text
https://github.com/Kiru-Sama/dsh-progress-reminder/releases/download/v3.0.0/dsh-progress-reminder-3.0.0.tgz
```

> 如果 GitHub 的下载域名被挡（`release-assets.githubusercontent.com` 连不上），
> 可以改用代理前缀，例如在地址前面加 `https://ghfast.top/`，或者直接把 tgz
> 下载下来、在“添加插件”里粘**本地文件的绝对路径**。

## 方式 C：本地目录

1. 把仓库解压/克隆到固定位置，比如 `C:\Users\你\Documents\dsh-progress-reminder`
2. `设置 → 插件 → 添加插件 → 从本地目录`
3. 选那个目录（选到能看到 `package.json` 的那一层）

管理器会自己把 `dsh-progress-reminder` 写进 profile 的依赖和
`dsh.profile.bundles`，不需要你手改 JSON。

> 旧版本文档写的是“手工 link”。现在桌面版插件管理器支持包名/tgz/本地目录，
> 手工 link 只保留给开发调试/多 profile 的旧流程，普通安装不用看。

## 常见安装错误

### `already-installed`

说明这个 profile 里已经有一份 `dsh-progress-reminder`，通常是旧的手工 link。
先按“移除旧 link”清掉，再重新添加：

```powershell
# 1) 从 profile 的 package.json 删掉 dsh-progress-reminder 的两处：
#    dependencies.dsh-progress-reminder
#    dsh.profile.bundles 里的 "dsh-progress-reminder"
# 2) 删掉旧链接（只删链接，不动源目录）
Remove-Item "$env:USERPROFILE\.dsh\profiles\desktop\node_modules\dsh-progress-reminder"
# 3) 完全退出 DSH，再重新打开，然后重新“从本地目录”添加
```

### `not-a-bundle`

检查包的 `package.json` 里是否有：

```json
"dsh": {
  "bundle": {
    "patch": "./cordis.patch.yml"
  }
}
```

并且 `cordis.patch.yml` 真的存在。当前版本已经带好，正常不会出现。

### `not-a-package`

选错目录了。要选**包根目录**（里面有 `package.json`），不是它的父目录，
也不是 `node_modules`。

## 重启并确认

**必须 `文件 → 退出` 完全退出再启动**，不是关窗口。新增 bundle 不会热加载。

重启后，侧栏底部（输入框左边那一列的最下面）会出现：

```
●  0/10 · 0秒                    梁文峰
```

点它展开面板，能看到步数/耗时/余额/时段，以及两个阈值输入框。

**如果没出现**：

1. `设置 → 插件` 里能看到 `dsh-progress-reminder` 吗？没看到说明第 2 步没装成功；
2. 看安装时管理器给的错误码：`already-installed` / `not-a-bundle` / `not-a-package`，
   分别按上面的排查；
3. 启动日志里搜 `did not activate`，有的话会点名哪个插件没激活。

## 手工 link（开发/旧版回退，普通用户不用）

只有这两种情况才需要：

- 你要改插件源码，希望改完直接生效；
- 你的 DSH 版本没有桌面插件管理器，只能用 `dsh plugin`。

### 0. 找到 profile

| 系统 | 默认位置 |
|---|---|
| Windows | `%USERPROFILE%\.dsh\profiles\desktop` |
| macOS / Linux | `~/.dsh/profiles/desktop` |

### 1. 登记到 profile 的 `package.json`

```jsonc
{
  "name": "dsh-profile-desktop",
  "private": true,
  "dependencies": {
    "dsh-progress-reminder": "link:C:/Users/你/Documents/dsh-progress-reminder"
  },
  "dsh": {
    "profile": {
      "bundles": [
        "@deepseek-ai/dsh-base",
        "@deepseek-ai/dsh-web-app",
        "dsh-progress-reminder"
      ]
    }
  }
}
```

`link:` 后面的路径用正斜杠 `/`。

### 2. 建链接

**Windows（junction，不需要管理员权限）：**

```powershell
$profile = "$env:USERPROFILE\.dsh\profiles\desktop"
New-Item -ItemType Directory -Force -Path "$profile\node_modules" | Out-Null
New-Item -ItemType Junction `
  -Path   "$profile\node_modules\dsh-progress-reminder" `
  -Target "C:\Users\你\Documents\dsh-progress-reminder"
```

**macOS / Linux：**

```bash
ln -s /Users/你/Documents/dsh-progress-reminder \
      ~/.dsh/profiles/desktop/node_modules/dsh-progress-reminder
```

### 3. 完全退出并重启

和桌面版安装一样，`文件 → 退出` 后重新打开。

## 配置

所有配置都有默认值，**不改也能用**。默认值在插件自己的
`cordis.patch.yml` 里，改那个文件后重启即可生效。

| 字段 | 默认 | 说明 |
|---|---|---|
| `stepThreshold` | `10` | 累计多少步提醒一次 |
| `timeThresholdSeconds` | `120` | 距上次汇报累计多少秒提醒一次 |
| `minGapSeconds` | `45` | 两次提醒最小间隔（兜底防连击） |
| `maxPerTurn` | `20` | 单回合提醒上限 |
| `exclude` | `["todo_write","ask_user_question"]` | 不计步的工具 |
| `includeSubagents` | `false` | 是否也管 subagent |
| `resetOnSpeech` | `true` | Agent 主动说话时归零计数 |
| `locale` | `zh` | 文案语言 `zh` / `en` |
| `balance.enabled` | `true` | 余额监控 |
| `balance.thresholdCny` | `1.5` | 低于多少提醒 |
| `balance.scope` | `total` | `total`=充值+赠送；`recharge`=只看充值 |
| `balance.checkSeconds` | `300` | 查余额间隔（最小 30） |
| `balance.repeatSeconds` | `1800` | 仍低额时多久再提醒；`0`=只提醒一次 |
| `tariff.enabled` | `true` | 显示峰谷时段 |
| `tariff.holidays` | `[]` | 中国法定节假日 `YYYY-MM-DD`，全天按低谷算 |

步数、秒数、余额阈值这三项**也能在侧栏面板里直接改，输入即生效**，不用重启。

> ⚠️ **同一个 profile 里不要放两行相同 id 的配置**。两份实例会各自注册
> `/progress-nudge/api` 前缀路由，重复路由会让整棵插件树启动失败。

## 更新 / 回滚

侧栏面板会提示 GitHub 上的新版本（`updateCheck`）。更新时三选一：

- **包名（推荐）**：`设置 → 插件` 里升级到新版号，或重新添加一次
  `dsh-progress-reminder`；也可以直接改版本号后重装：

  ```powershell
  # 在 profile 目录里
  pnpm add dsh-progress-reminder@latest
  ```

- **Release tgz**：在“添加插件”里重新粘贴 latest 地址。若提示 `already-installed`，
  先在 `设置 → 插件` 移除旧版，再添加新版，然后完全退出重启。
- **本地目录**：覆盖原目录（`git pull` / 解压新版），`文件 → 退出` 后重启。

回滚到指定版本（走 npm 最省事）：

```powershell
pnpm add dsh-progress-reminder@3.0.0
```

或者用带版本号的 Release 地址：

```text
https://github.com/Kiru-Sama/dsh-progress-reminder/releases/download/vX.Y.Z/dsh-progress-reminder-X.Y.Z.tgz
```

把 `X.Y.Z` 换成要回滚的版本；该版本必须已经发过 Release 或 npm。

> **配置放在哪**：默认值在**插件包内**的 `cordis.patch.yml` 里，**不在** profile 的
> `cordis.patch.yml`。想改长期生效的默认值，就改插件目录里那份（重启生效）——
> 但**更新/重装插件会覆盖它**，改之前先备份。
>
> ⚠️ 侧栏面板里调的值**只对当前运行有效**：它存在内存里，重启就回到
> `cordis.patch.yml` 的值。要长期生效必须改文件。

## 卸载 / 回滚

### 桌面版

`设置 → 插件` 里移除 `dsh-progress-reminder`，然后 `文件 → 退出` 重启。

### 手工 link / 管理器不可用

1. 从 profile 的 `package.json` 里删掉两处：`dependencies` 与 `dsh.profile.bundles`；
2. 删掉 `<profile>/node_modules/dsh-progress-reminder`（junction 用 `Remove-Item`，
   只删链接、不动源目录）；
3. 重启 DSH。

插件不改动 DSH 本体任何文件，删掉即彻底干净。

## 已知限制

- **提醒只能在步骤边界投递**。工具执行期间 agent 循环是阻塞的，模型读不到任何东西；
  所以一个跑 5 分钟的命令，提醒要等命令返回才发得出去。跨越阈值但还没到边界时，
  界面会用「已到期 N 秒 · 等当前步骤结束后提醒」明确告诉你它在等。
- **余额依赖宿主登录**。余额读的是 DSH 自己的 `deepseekAccount` 服务（设置页
  「账号与余额」显示的那个）。没登录、或用第三方中转站时，面板会显示「不可用」，
  不会报错也不会瞎编数字。
- **峰谷时段是 DeepSeek 官方 API 的计费口径**（工作日北京时间 09:00–12:00、
  14:00–18:00 为高峰，其余半价）。**中国法定节假日无法离线判断**，需要手工填
  `tariff.holidays`，否则只按周末 + 时段算，面板上会注明这一点。
- 目前只在 **Windows + DSH 桌面版**上实测过。macOS / Linux 理论上可用（手工 link
  用 symlink），但未验证。
- 只在 `desktop` profile 下实测；多 profile 场景未验证。
- **侧栏面板调的阈值不持久**。改完立即生效，但重启回到插件包内 `cordis.patch.yml`
  的值。要固定下来就改那个文件（会被更新覆盖，先备份）。
- **版本检测需要能连 `api.github.com`**。如果本机挂了改写 hosts 的加速工具
  （Steam++ / Watt Toolkit 之类），面板会显示「查询失败（unable to verify…）」——
  这是刻意不绕过证书校验的结果，不是 bug。
