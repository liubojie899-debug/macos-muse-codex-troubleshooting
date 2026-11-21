# Muse / Codex macOS 连接故障排查完整指南（单文件版）

> 本文件是资料包的**单文件完整版**，可单独阅读；文内跳转均指向本文件的章节锚点，不依赖其它本地文件。  
> 配套 ZIP 中另有分章 Markdown（`README.md` 与 `docs/`）以及 `MANIFEST.txt`、`SHA256SUMS.txt`，便于按章节查阅和校验。  
> 记录日期：**2026-11-21**，时间均为**北京时间（Asia/Shanghai，UTC+8）**。两种修复均已由用户确认，但都只是单案例结果，不是官方指南。

## 目录

- [macOS 上 Muse / Codex 连接故障（单案例公开资料包）](#chapter-1)
- [Muse：系统 HTTP 代理冲突（单案例已验证修复）](#chapter-2)
- [Codex：为 `chatgpt.com` 新增域名规则（单案例已验证修复）](#chapter-3)
- [排障时间线与证据对照](#chapter-4)
- [常见问题与判断陷阱](#chapter-5)
- [隐私要求与脱敏说明模板](#chapter-6)

---

<a id="chapter-1"></a>

## macOS 上 Muse / Codex 连接故障（单案例公开资料包）

> 这是 **2026-11-21** 在一台 macOS 机器上整理的排查记录，时间均为 **北京时间（Asia/Shanghai，UTC+8）**。  
> 两处修复均已由用户确认有效，但都只是**单案例**结果，不是官方指南，也不保证适用于所有 Shadowrocket 规则、版本或网络环境。

本文件可单独阅读并按步骤对照。更完整的证据、时间线、FAQ 与脱敏模板在 `docs/` 中，并全部收入配套 ZIP。

## 当前状态

| 应用 | 状态 | 本案例已验证的做法 |
|------|------|-------------------|
| Muse | **已由用户于 13:06 确认恢复** | 将 Shadowrocket「代理类型」从 HTTP 改为 None（界面明示 TUN 模式）。见下方「修复一」与 [docs/muse-system-proxy.md](#chapter-2)。 |
| Codex | **已由用户于 14:26 确认解决** | 在不重连 VPN、不切换配置的前提下，新增规则 `DOMAIN-SUFFIX,chatgpt.com,PROXY`。此处 `PROXY` 只是当时编辑页预填的、本机已有策略名，其他人应选择**自己配置里已存在且可用的策略**。见下方「修复二」与 [docs/codex-chatgpt-domain-rule.md](#chapter-3)。 |

不要把两处修复混成一步：改成 None / TUN 后，Muse 已恢复，但 Codex 当时仍失败；Codex 是在后来新增域名规则后才被确认解决的。

## 你可能看到的症状

- **Muse**：无法正常连接。本案例中，相关请求经过系统有效 HTTP 代理 `127.0.0.1:1082`，并在 TLS 握手前被重置。
- **Codex**：界面曾依次出现 *Couldn’t load workspace settings*（无法加载工作空间设置）、*Starting chat* 后无回复、*Reconnecting* 直至 5/5，以及 *workspace routing discovery timed out*（工作空间路由发现超时）。

本包以「应用可正常使用，并且 Codex 消息能收到回复」为恢复标准。

## 已知环境（仅列出已核对的版本）

- 操作系统：macOS
- Shadowrocket（标注 iOS 兼容）：**2.2.93**
- Muse：**12.1**（build **1114886626**）。代码签名身份为 Meta，Gatekeeper 接受（notarized）。
- Codex 桌面应用：**26.1117.41037**（build **29803**）。代码签名身份为 OpenAI，Gatekeeper 为 accepted。深度签名校验遇到 Permission denied，**不能**表述为完整签名验证通过。

`127.0.0.1:1082` 是本机回环代理地址与本地端口；`198.18.0.0/15` 是隧道 / fake-IP 一类诊断机制地址段。它们都**不是**公网出口 IP。

## 修复一：Muse——系统 HTTP 代理改为 None（TUN）

只改「代理类型」这一项。不要顺便关闭 VPN / TUN，也不要同时改 DNS、退出登录、清缓存或重装。这项修复**不应**写成「所有人都应该关掉系统代理」。

1. 打开 Shadowrocket 设置，进入「代理」，找到「代理类型」。
2. 本案例当时勾选的是 **HTTP**。将其改为 **None**。
3. 界面会明示这是 **TUN 模式**。确认后不要再改其它项。

**本案例验证：** 有效系统 HTTP/HTTPS 代理消失，VPN / TUN 仍保留；Muse 新日志出现 HTTPS 200；用户于 **13:06** 确认恢复。全程没有退出登录、清缓存或重装。

**回滚：** 若出现断网、其它依赖该 HTTP 代理的应用异常，或只是要恢复变更前的状态，将「代理类型」改回 **HTTP**，其它项保持不动。改回 HTTP 只为恢复原状态，**不建议**为了复现故障再改回去。

完整前置条件、风险与只读确认见 [docs/muse-system-proxy.md](#chapter-2)。

## 修复二：Codex——新增 `chatgpt.com` 的域名规则

先记录原值并备份你将要改动的那份配置。不要用订阅更新覆盖本地刚改过的规则。一次只加这一条，不要同时重连 VPN、切换配置或改其它规则。

本案例的实际操作是：

1. 在请求详情中选择「添加域名到规则」。
2. 当时真实编辑页的预填值是：类型 **DOMAIN-SUFFIX**，策略 **PROXY**，域名为 `ab.chatgpt.com`，「扩展匹配」和「预匹配」均为 **Off**。
3. **只把域名改成 `chatgpt.com`**，其余预填值一律不改，然后保存。
4. 最终新增的规则是：`DOMAIN-SUFFIX,chatgpt.com,PROXY`。
5. 没有重连 VPN，也没有改用其它配置。用户于 **14:24** 确认保存完成。

**关于策略名：** 这里的 `PROXY` 只是本案例编辑页里已经预填、且当时存在的策略。其他人应选择自己配置中**已经存在并且确认可用**的策略，不必照抄这个名字。

**本案例验证（可核验的部分）：**

- 新增规则前，对公共目标 `chatgpt.com` 的无认证 curl 在 **14:19:54–14:19:59** TLS 超时约 5 秒，HTTP 状态为 **000**，curl 退出码 **28**。HTTP 000 表示没有收到 HTTP 响应，**不是**服务器返回了状态码 000。
- 保存规则后，对**同一公共目标**的无认证 curl 在 **14:25:00–14:25:02** 证书校验通过，HTTP **200**，curl 退出码 **0**；TLS 约 1.03 秒，总耗时约 1.57 秒。
- 同期 Codex 的 `account/read` / `getAuthStatus` 多次在约 2–8 ms 内返回，且 `errorCode=null`。响应过快，**可能来自缓存，不能单独当作修复成功的证据**。
- 用户于 **14:26** **明确确认问题已解决**。

**尚未核验、因此不能写进「已证实」的部分：** 本包没有「新规则已被命中，且策略为 PROXY」的最终请求日志截图，所以**不能**声称已经用运行日志核验了这条新规则的优先级或命中结果。新增规则也不保证一定覆盖其它冲突规则；是否生效要以保存后的新请求日志为准。

**回滚：** 只删除本次新增的这一条规则，不要顺手改其它规则或整份重置。删除后如需对照，应重新检查目标站点请求，而不是假定会立即回到某一次历史状态。

完整证据、无效尝试与规则编辑页说明见 [docs/codex-chatgpt-domain-rule.md](#chapter-3)。

## 务必处理：临时日志

若排查时打开过 Shadowrocket 的临时请求日志或同类开关，**采集完证据后务必关闭**。本案例在整理资料时**尚未确认**该开关已经关闭，因此这里不声称「日志已关闭」。请在本机自行确认。

## 不要踏入的判断陷阱（摘要）

详细说明见 [docs/faq.md](#chapter-5)。

- 没有 401 / 403 或封禁文案，不等于当时已证明账号被封，也不等于当时已证明账号完全健康。本案例最终是在调整本机规则后由用户确认恢复。
- 只换节点，并不能自动消除运行日志里已经出现的 `DIRECT` 命中。节点点选在本案例中**没有已验证的改善**；且客户端顶部节点名与选中标记曾不一致，**实际出口未经核验**。节点只用「节点 A / 节点 B」泛称，不能写成「两个国家的出口都失败」。
- Codex 文案里的 *workspace routing* 指工作空间路由发现，**不是** VPN 分流里的 routing。
- curl 的 HTTP **000** 不是服务器状态码。
- 看到 `198.18.0.0/15` 这类 fake-IP / 隧道地址，本身不能单独证明故障，也不能把它当成公网出口。
- 其它域名或其它应用能访问，不代表 `chatgpt.com` 或 Codex 的目标一定可达。
- 运行中的请求日志与配置编辑页里搜到的规则不一致时，机制往往尚未证实。本案例遇到过这种现象，**不要轻易归因于「缓存」或「软件 bug」**。

## 文件列表

| 文件 | 内容 |
|------|------|
| [README.md](#chapter-1) | 本文件：总览与两种已验证修复（可单独使用） |
| [Muse-Codex-macOS-troubleshooting.md](#) | 单文件完整版，便于单独下载 |
| [docs/muse-system-proxy.md](#chapter-2) | Muse 修复的症状、前置条件、步骤、验证、风险与回滚 |
| [docs/codex-chatgpt-domain-rule.md](#chapter-3) | Codex 修复的症状、步骤、验证边界、回滚与无效尝试 |
| [docs/timeline-and-evidence.md](#chapter-4) | 当日排障时间线与证据对照 |
| [docs/faq.md](#chapter-5) | 常见判断陷阱 |
| [docs/privacy-and-reporting.md](#chapter-6) | 隐私清单与可复制的脱敏说明模板 |
| MANIFEST.txt | 文件清单 |
| SHA256SUMS.txt | 文本文件的 SHA256 |

本包不含执行脚本、不含真实配置或数据库、不含原始截图或聊天内容，也不附加许可证授权。

## 公开来源（请区分性质）

社区里出现过相近的工作空间发现失败，只能当同症状参考，**不是本案例的官方根因，也不能直接当成可套用的修复**：

- [openai/codex#47145](https://github.com/openai/codex/issues/47145)（Windows）
- [openai/codex#53236](https://github.com/openai/codex/issues/53236)（Windows）
- [openai/codex#49443](https://github.com/openai/codex/issues/49443)（macOS）

[openai/codex#45529](https://github.com/openai/codex/pull/45529) 是工作空间发现功能的说明性变更，**不是一条修复记录**。

Shadowrocket 模式的社区说明（例如 [这份社区 README](https://github.com/LOWERTOP/Shadowrocket/blob/d009b958/README.md)）只作背景阅读，**不是厂商保证**。

官方支持入口：[https://help.openai.com/](https://help.openai.com/)。不要使用可能已经失效的旧客服文章链接。

## 上传前请再做一次隐私检查

不要把邮箱、真实姓名、用户名、机器名、设备 ID、账号 / 工作空间 ID、订阅链接、服务商或节点国家名、代理公网出口 IP、聊天原文、原始截图、绝对个人路径或真实配置 / 数据库写进公开仓库。详细清单和脱敏模板见 [docs/privacy-and-reporting.md](#chapter-6)。


---

<a id="chapter-2"></a>

## Muse：系统 HTTP 代理冲突（单案例已验证修复）

> 适用范围：这是一台 macOS 机器上、已由用户于 **2026-11-21 13:06（北京时间）** 确认恢复的记录。不是对所有 Shadowrocket 模式、版本或网络环境的保证。  
> 不要把这项修复理解成「所有人都应该关掉系统代理」。同一变更**没有**同时修好当时的 Codex。

## 结论

在本案例中，Muse 无法正常连接，与系统有效 HTTP/HTTPS 代理指向 `127.0.0.1:1082` 有关。将该代理类型从 HTTP 改为 None（界面明示 TUN 模式）后：

- 有效的系统 HTTP/HTTPS 代理消失；
- VPN / TUN 仍然保留；
- Muse 新日志中出现 HTTPS 200；
- 用户确认 Muse 已恢复正常。

修复过程中没有退出登录、清理缓存或重装应用。已核对其代码签名身份为 Meta，Gatekeeper 接受（notarized）。无需、也不应通过绕过签名来处理这个问题。

`127.0.0.1:1082` 是本机回环地址和本地端口，不是代理服务器的公网 IP。

## 前置条件（本案例实际具备）

请先对照，而不是直接改设置：

- 操作系统为 macOS。
- 使用 Shadowrocket（标注 iOS 兼容），本案例版本为 **2.2.93**。
- Muse 版本为 **12.1**（build **1114886626**）。
- 出问题期间，系统**有效** HTTP/HTTPS 代理为 `127.0.0.1:1082`。
- 常规网络服务里的代理开关**可以已经是关闭的**，但有效代理仍可能是开启的。本案例中，有效代理来自动态 Network Extension 配置。因此：不要只根据网络服务面板判断「系统代理已关」；**不要盲目写入 `scutil` 或其它系统代理配置**。
- 已核对其代码签名身份为 Meta，Gatekeeper 接受（notarized）。本文件不提供任何绕过签名的做法。
- 当时需要保留现有登录态。本案例证明：至少在这台机器上，不必退出账号也能完成该项对照。

## 症状与证据

- Muse 无法正常使用（连接失败）。
- 应用日志表明相关请求经过本地 `1082` 代理。
- 经该本地 HTTP 代理时，连接在 TLS 握手前被重置（RST）。
- 对同一本地代理做显式代理请求（curl 走该代理）失败。
- 改为 None（TUN 模式）后，当时路径上对公开相关主机可以完成 TLS / HTTP；Muse 新日志出现 HTTPS 200，用户于 13:06 确认恢复。

本文件不附原始日志、截图或聊天内容。

## 一步步操作

只改下面这一项，其它开关保持不动，尤其不要顺便关闭 VPN / TUN，也不要同时改 DNS、退出登录、清缓存或重装。

1. 打开 Shadowrocket 设置，进入「代理」，找到「代理类型」。
2. 当时勾选的是 **HTTP**。将其改为 **None**。
3. 界面会明示这是 **TUN 模式**。确认后不要再改其它项。
4. 观察本案例已出现的结果：有效 HTTP/HTTPS 代理消失，VPN / TUN 仍在运行，Muse 在不退出登录的前提下恢复。

这是一次可回滚的对照，不是「把代理和 VPN 全部关掉」的修复。

## 验证

本案例中的验证是：变更后 Muse 新日志为 HTTPS 200，并且用户明确确认可以正常使用。不要只根据其它无关网站可用来判断 Muse 已恢复。

## 风险

- 取消系统 HTTP/HTTPS 代理后，其它依赖该代理的应用可能改走不同路径，出现能用、不能用或行为变化。Muse 恢复，不表示所有应用都会同样受益。Codex 在同一安排下当时仍失败，就是反例。
- 若对照时同时关闭 VPN、改 DNS、退出登录或清理数据，将无法判断是哪一步起了作用，也可能造成不必要的登录损失。
- Shadowrocket 的规则、模式和版本不同，界面文案或实际效果可能不同。2.2.93 上的结果不能外推为所有版本都成立。
- 社区对 Shadowrocket 模式的说明（例如 [一份社区 README](https://github.com/LOWERTOP/Shadowrocket/blob/d009b958/README.md)）只能当背景阅读，不是厂商保证，也不能代替本机对照。

## 回滚

若出现断网、其它关键应用不可用，或只是要回到变更前的状态，将「代理类型」改回 **HTTP**，其它项保持不动。

**改回 HTTP 只为恢复原状态，不建议为了复现故障再改回去。**

回滚后是否立刻回到变更前的全部网络行为，需要在本机确认，不要假定一条命令可以通用复位。

## 建议的只读确认（可选）

若只想确认有效系统代理是否仍指向 `127.0.0.1:1082`，可以使用系统自带的只读查询。下面命令在本案例中用于观察有效代理，但：

> **建议只读检查，未作为所有平台通用验证。** 不要向配置写入任何值；不要把它做成修复脚本。

```bash
scutil --proxy
```

关注有效的 HTTP / HTTPS 代理主机与端口是否为 `127.0.0.1` 和 `1082`。常规网络服务代理显示为关闭时，这里仍可能显示有效代理已开启。

本包不提供一键诊断或一键修复脚本。

## 明确不建议的做法

- 不要为了「修好 Muse」同时关闭 VPN / TUN 或改 DNS。
- 不要盲目用 `scutil` 或其它方式写入系统代理。
- 不要把绕过应用签名、修改应用完整性或重打包当作步骤。
- 不要在公开说明里写入订阅链接、服务商域名、节点密钥、账号或个人标识。
- 不要声称「所有走系统代理的应用改成 None 都能修好」。


---

<a id="chapter-3"></a>

## Codex：为 `chatgpt.com` 新增域名规则（单案例已验证修复）

> 适用范围：这是一台 macOS 机器上、已由用户于 **2026-11-21 14:26（北京时间）** 确认解决的记录。不是官方修复，也不保证新增规则在所有配置里都会覆盖冲突项。  
> Codex 桌面应用版本仅核对此案的 **26.1117.41037**（build **29803**）。

## 结论

在 Muse 已恢复、系统有效 HTTP 代理已消失之后，Codex 仍失败。随后在请求详情中新增规则：

```text
DOMAIN-SUFFIX,chatgpt.com,PROXY
```

保存后，同一公共目标 `chatgpt.com` 的无认证 curl 从 TLS 超时（HTTP 000，curl 28）变为证书校验通过且 HTTP 200（curl 0）。用户明确确认问题已解决。

必须同时记住下面的边界：

- 这里的 `PROXY` 只是本案例编辑页预填的、当时已有策略名。其他人应选择自己配置里**已经存在并且确认可用**的策略，不必照抄该名字。
- 本包**没有**「新规则已被命中为 PROXY」的最终请求日志截图，因此不能把规则优先级或命中结果写成已核验事实。是否覆盖其它规则，只能看保存后的新日志。
- 同期 Codex 的 `account/read` / `getAuthStatus` 曾多次在约 2–8 ms 内返回且 `errorCode=null`，速度过快，**可能来自缓存，不能单独当作成功证据**。本案例把用户确认与公共目标 TLS / HTTP 恢复作为主要验证。

## 前置条件与环境

- 操作系统为 macOS；Shadowrocket（标注 iOS 兼容）**2.2.93**。
- 当时已处于 Muse 恢复后的网络安排：代理类型为 None（TUN 模式），有效系统 HTTP/HTTPS 代理已不存在，VPN / TUN 仍保留。因此不能再把后来的 Codex 失败简单写成「仍在走本地 `1082` HTTP 代理」。
- 正在使用的配置在界面上带有「正在使用」标记（蓝勾）。本案例中该配置显示名为 `default.conf`。模块列表当时为空。
- 已核对其代码签名身份为 OpenAI，Gatekeeper 为 accepted。深度签名校验遇到 Permission denied，不能表述为完整签名验证通过。本文件不讨论绕过签名。
- 操作前应备份或记录原值。不要让随后的订阅更新覆盖这次本地新增的规则。
- 本案例没有重连 VPN，也没有改用其它配置。

## 用户可见症状

按观察顺序，界面曾出现：

1. *Couldn’t load workspace settings*（无法加载工作空间设置）
2. 正式更新并更换主进程后，主界面可以出现，但新对话停在 *Starting chat*，长时间没有回复。这不能算恢复。
3. *Reconnecting* 直至 **5/5**。界面上的“思考中”一类状态不能证明连接正常。
4. *workspace routing discovery timed out*（工作空间路由发现超时）

这里的 *workspace routing* 指 Codex 的工作空间路由发现，不是 VPN 客户端里的分流 routing。详见 [faq.md](#chapter-5)。

更早的应用侧记录（修复前）还包括：`account/read` 曾等待约 10881 ms 和 25869 ms 后失败，错误码 `-32603`；`config/read` 较快，随后出现 Workspace routing unavailable。已查日志中没有作为本案依据的 401 / 403 或明确封禁文案，但这在当时也不足以证明账号完全健康。部分日志中的 `status 500` 是本地包装，不能写成服务端 HTTP 500。

## 与规则有关的运行观察

在新增规则之前，请求日志里曾经出现过：

- `chatgpt.com:443` 命中 `DOMAIN,chatgpt.com,DIRECT`
- `ab.chatgpt.com` 命中 `DOMAIN-SUFFIX,chatgpt.com,DIRECT`

同一时期，在当前正在使用的 `default.conf` 编辑页里搜索 `chatgpt.com`，**没有搜到结果**；模块列表为空。

运行中的命中记录与编辑页里能搜到的规则不一致，这一现象的机制**尚未证实**。不要把它归因于「缓存」或「软件 bug」。本文只如实记录这个矛盾，并据此强调：必须以保存规则之后的新请求日志为准，而不是只根据编辑页搜索结果下结论。

另外，曾尝试只改用同一份 `default.conf`、**不改规则**。该动作之后，在 14:09 和 14:19 对公共目标的 TLS 仍然失败，因此**不是一次有效修复**。

## 一步步操作（本案例实际执行）

先记录编辑页上的原值。若你的预填内容与下面不同，不要为了「对齐教程」去改那些未经验证的字段。

1. 打开相关请求的「请求详情」，选择「添加域名到规则」。
2. 本案例真实编辑页的默认值是：
   - 类型：`DOMAIN-SUFFIX`
   - 策略：`PROXY`（这是当时已有策略的示例名）
   - 域名：`ab.chatgpt.com`
   - 扩展匹配：Off
   - 预匹配：Off
3. **只将域名改为 `chatgpt.com`**，其余项目保持不动，然后保存。
4. 最终新增：`DOMAIN-SUFFIX,chatgpt.com,PROXY`。
5. 不要重连 VPN，不要切换或重新导入配置，也不要同时改其它规则。

将域名从预填的 `ab.chatgpt.com` 改为 `chatgpt.com`，是为了按 `DOMAIN-SUFFIX` 的通常含义覆盖该域及其子域。若你使用的不是 `DOMAIN-SUFFIX`，不要直接套用这一改法。

## 验证

本案例核验了下面这些事实，请不要自行补写成未出现过的「已命中 PROXY」日志：

| 时间（北京时间） | 对公共目标 `chatgpt.com` 的无认证 curl | 说明 |
|------------------|----------------------------------------|------|
| 14:19:54–14:19:59 | TLS 超时约 5 秒，HTTP **000**，curl 退出码 **28** | 没有拿到 HTTP 响应。000 不是服务器状态码。 |
| 14:25:00–14:25:02 | 证书校验通过，HTTP **200**，curl 退出码 **0**；TLS 约 1.03 秒，总耗时约 1.57 秒 | 与保存新规则之后的时间窗口一致。 |
| 14:26 | 用户明确确认问题已解决 | 这是应用侧恢复的主要依据。 |

同期 Codex `account/read` / `getAuthStatus` 多次约 2–8 ms 且 `errorCode=null`，可能是缓存，不能单独作为成功证据。

其它域名或其它应用可达，不能代替对 `chatgpt.com` 以及 Codex 本身的检查。

## 回滚

只删除**本次新增的那一条**规则，不要清空整份规则列表，也不要用订阅更新去覆盖本地其它修改。

删除后应重新检查目标请求。回滚的目的是恢复新增前的配置，不是为了主动复现故障。

## 无效或未证实有改善的尝试

列出这些是为了减少重复劳动，不是建议再做一遍：

1. **只完成 Muse 那次「代理类型改为 None / TUN」**  
   Muse 已恢复，当时 Codex 仍失败。
2. **关窗口或重新唤起，未验证应用真正退出；随后实际更新更换了进程，仍报错**  
   首次所谓「重开」不能当成一次已证实的完整退出。后来的实际更新确已更换进程，但错误仍在。
3. **在保留 None 模式的前提下点选更换节点**  
   没有已验证的改善。客户端首页顶部节点名与列表选中标记还曾经不一致，实际出口未经证实。因此不能把实验写成「两个国家或两家运营商的出口都失败」。节点只用「节点 A / 节点 B」泛称。用户后来表示已恢复原来的节点，但真实流量路径仍未证实。
4. **只改用同一份正在使用的配置、不新增或不修改规则**  
   本案例在 14:06 只使用同一份 `default.conf` 且不改规则，14:09 与 14:19 的 TLS 仍然失败。这不是有效修复。

没有把退出账号、清理缓存、重装应用，或同时关闭 VPN / DNS 当作已执行的修复步骤。

## 社区同症状（不是本案例的官方修复）

下列讨论出现过相近的工作空间发现失败，只能对照症状。不要把其中建议直接当成适用于 26.1117.41037 的通用修复，也不要假定那些报告与本案例有相同根因：

- [openai/codex#47145](https://github.com/openai/codex/issues/47145)（Windows）
- [openai/codex#53236](https://github.com/openai/codex/issues/53236)（Windows）
- [openai/codex#49443](https://github.com/openai/codex/issues/49443)（macOS）

[openai/codex#45529](https://github.com/openai/codex/pull/45529) 是工作空间发现功能的说明性变更，不是一条修复记录。

官方支持入口：[https://help.openai.com/](https://help.openai.com/)。

## 临时日志

若为了查看「请求详情」而打开过临时日志，采集证据后务必关闭。整理本包时，**尚未确认**该开关已经关闭，因此不声称「已经关闭」。请在本机自行检查。


---

<a id="chapter-4"></a>

## 排障时间线与证据对照

时间均为 **2026-11-21，北京时间（Asia/Shanghai，UTC+8）**。这里只保留对后来人有判断价值的节点，不堆砌全部试验细节。不附原始截图、聊天原文或完整日志。

## 时间线

| 时间 | 做了什么 / 看到了什么 | 当时能得出的结论 |
|------|------------------------|------------------|
| 13:06 前 | Muse 走系统有效 HTTP 代理 `127.0.0.1:1082`，TLS 握手前被重置；显式走该代理的请求失败。 | Muse 故障与该系统 HTTP 代理路径相容。 |
| 约此前 | 将 Shadowrocket「代理类型」从 HTTP 改为 None（TUN）。有效 HTTP/HTTPS 代理消失，VPN / TUN 保留。 | 单项对照开始。 |
| 13:06 | Muse 新日志出现 HTTPS 200，用户确认 Muse 恢复。 | **Muse 单案例修复成立。** 当时不要据此宣布 Codex 已修复。 |
| 13:06 之后、新增规则之前 | Codex 仍出现 *Couldn’t load workspace settings*，随后出现 Starting chat、Reconnecting 5/5，以及 workspace routing discovery timed out。期间曾有关窗口或重新唤起，但未验证应用真正退出；随后实际更新更换了进程，仍报错。 | None / TUN 不足以修复本案 Codex。 |
| 修复前的应用记录 | `account/read` 约 10881 ms、25869 ms 后失败，错误码 `-32603`；`config/read` 较快，随后 Workspace routing unavailable。已查日志无 401 / 403 或明确封禁文案；部分 `status 500` 为本地包装。 | 不能写成「已证实封号」，也不能写成「账号已证实完全健康」，更不能写成「服务端 HTTP 500」。 |
| 修复前的公共目标对照 | 对 `chatgpt.com` 的路径曾落入 `198.18.0.0/15`（本案例观察到 `198.18.0.22`）并经 `utun` 转发：TCP 可建立，TLS 以 EOF 结束，curl 退出码曾为 35。后一次对照见 14:19。 | 与连接路径问题相容，但 fake-IP 本身不单独证明故障，也不能代替工作空间请求的目标证据。 |
| 换节点对照 | 保留 None 模式点选其它节点，没有已验证的改善；顶部节点名与选中标记曾不一致。 | **实际出口未核验**，不能写成两个国家出口都失败。节点只用 A / B 泛称。 |
| 13:49 | 请求日志中，`chatgpt.com:443` 命中 `DOMAIN,chatgpt.com,DIRECT`。 | 运行态曾把该主机导向 DIRECT。这不能单独解释编辑页稍后搜不到规则的原因。 |
| 14:06 | 只使用同一份正在使用的 `default.conf`，不改规则。 | 这一步本身还不能叫修复。 |
| 14:09、14:19 | 对公共目标的 TLS 仍然失败。 | **仅改用同一配置、不改规则，不是有效修复。** |
| 14:19:54–14:19:59 | 对 `chatgpt.com` 的无认证 curl：TLS 超时约 5 秒，HTTP 000，curl 28。 | 保存新规则前的对照基线。 |
| 14:21 | 请求日志中，`ab.chatgpt.com` 命中 `DOMAIN-SUFFIX,chatgpt.com,DIRECT`。 | 运行态再次出现指向 DIRECT 的后缀规则命中。 |
| 同期编辑页 | 当前正在使用的 `default.conf`（蓝勾）中搜索 `chatgpt.com` 无结果，模块列表为空。 | 运行命中与编辑页可见规则不一致。**机制未证实，不要归因于缓存或软件 bug。** |
| 14:23–14:24 | 从「请求详情」→「添加域名到规则」进入编辑页。预填为 DOMAIN-SUFFIX、PROXY、`ab.chatgpt.com`，扩展匹配 / 预匹配 Off。只把域名改为 `chatgpt.com` 后保存。最终新增 `DOMAIN-SUFFIX,chatgpt.com,PROXY`。未重连 VPN，未切换配置。用户 14:24 确认完成。 | 这是本案实际执行的单项变更。`PROXY` 只是当时已有策略的示例名。 |
| 14:25:00–14:25:02 | 对同一公共目标的无认证 curl：证书校验通过，HTTP 200，curl 0；TLS 约 1.03 秒，总耗时约 1.57 秒。 | 公共目标连通性已改善。**没有**最终「新规则命中 PROXY」的截图，故不把命中情况写成已核验。 |
| 同期 | Codex `account/read` / `getAuthStatus` 多次约 2–8 ms，`errorCode=null`。 | 可能是缓存，不能单独作为成功证据。 |
| 14:26 | 用户明确确认问题已解决。 | **Codex 单案例修复由用户确认成立。** |
| 资料整理时 | 临时日志是否已关闭，尚未确认。 | 文档要求采集证据后务必关闭，但不声称已经关闭。 |

## 证据对照（保存规则前后）

针对同一公共目标 `chatgpt.com`、无认证 curl：

| 项目 | 保存新规则前（14:19:54–14:19:59） | 保存新规则后（14:25:00–14:25:02） |
|------|-----------------------------------|-----------------------------------|
| TLS | 超时约 5 秒 | 约 1.03 秒，证书校验通过 |
| HTTP | 000（无响应，不是服务器状态码） | 200 |
| curl 退出码 | 28 | 0 |
| 总耗时 | 受 5 秒超时限制 | 约 1.57 秒 |

应用侧最终依据是用户于 14:26 确认已解决，而不是单独的毫秒级 `errorCode=null`。

## 对后来人最有用的判断点

- 先分清两件事：系统 HTTP 代理影响了 Muse；Codex 在代理关闭后仍失败，后来与 `chatgpt.com` 被运行态导向 `DIRECT` 的观察放在同一条排查线上。
- 不要用「配置编辑页搜索无结果」否定请求日志里已经出现的命中，也不要反过来编造机制。
- 不要把「换到正在使用的那份配置」当成改规则。
- 验证要看同一目标在变更前后的对比，加上应用是否真正恢复；不要用其它网站能打开来代替。


---

<a id="chapter-5"></a>

## 常见问题与判断陷阱

## 这是账号被封吗？

本案例**没有**把已查日志中的 401、403 或明确停用 / 封禁文案当作依据，因此不能把当时的超时报错写成封号通知。反过来，当时也没有足够证据宣称账号完全健康。

后来在本机新增 `chatgpt.com` 规则后，公共目标的 TLS / HTTP 恢复，用户确认 Codex 已解决。这说明本案最终被确认的恢复，是与本机规则 / 连接路径调整放在一起的，而不是一次「证实封号后再解封」的过程。

若你的界面或邮件里出现了明确的账号停用说明，应以官方说明为准，而不是套用本包。

## 为什么换节点不能解决已经出现的 DIRECT 命中？

节点决定的是「走哪一条代理出口」，规则命中 `DIRECT` 表示该请求按直连处理。如果运行日志里 `chatgpt.com` 已经命中 `DIRECT`，只换节点通常不会自动改变这条命中。

本案例中，点选更换节点没有已验证的改善。再加上客户端顶部节点名与列表选中标记曾经不一致，**实际出口未经核验**，所以不能把结果写成「两个国家的出口都失败」。节点只以「节点 A / 节点 B」泛称。

## workspace routing 是 VPN routing 吗？

不是。Codex 提示里的 *workspace routing discovery timed out* 指应用在发现或读取工作空间路由信息时超时，不是 Shadowrocket 里「这条流量走哪条代理策略」的 routing。两者可能同时出问题，但不能把两个词当成一回事。

## HTTP 000 是服务器返回了 000 吗？

不是。本案例中 curl 记录的 HTTP **000** 出现在 TLS 超时、没有拿到 HTTP 响应的时候。它表示客户端没有得到有效的 HTTP 状态码，不是服务器主动返回了状态 000。对应的 curl 退出码 28 表示超时。

同样，本案部分应用日志里的 `status 500` 是本地包装，也不能直接写成服务端 HTTP 500。

## 看到 fake-IP（例如 198.18.0.0/15）就证明出故障了吗？

不能单独这么证明。`198.18.0.0/15` 常见于隧道 / fake-IP 机制，用来在本地标识被代理的目标，**不是公网出口 IP**。本案例在修复前确实观察过公共目标落入该地址段，并出现 TCP 成功但 TLS 失败；这与路径问题相容，但「出现 fake-IP」本身只说明流量走了这类机制，不能单独当作故障证据，更不能把它写成运营商地址。

## 其它域名或其它应用能访问，能说明目标也可达吗？

不能。本案例中，Muse 恢复之后 Codex 仍失败；部分其它连接正常，也不代表 `chatgpt.com` 或 Codex 所用目标当时可达。验证必须针对你真正要修复的那个目标，以及应用本身是否恢复。

## 编辑页搜不到规则，但请求日志里有命中，这是缓存或软件 bug 吗？

本案例确实观察过：运行日志出现 `DOMAIN,chatgpt.com,DIRECT` 和 `DOMAIN-SUFFIX,chatgpt.com,DIRECT`，但正在使用的配置编辑页搜索 `chatgpt.com` 无结果。这个矛盾的机制**尚未证实**。不要在原因未核对时写成「缓存问题」或「软件 bug」。比较稳妥的做法是：记录两边的观察，变更后以新的请求日志和应用表现为准。

## 新增一条规则就一定能覆盖原来的 DIRECT 吗？

不保证。规则优先级和实际命中要以保存后的新日志为准。本案例没有最终的新 `PROXY` 命中截图，因此没有把「新规则已被命中」写成已核验事实。用户确认恢复，加上同一公共目标从 TLS 超时变为 HTTP 200，是本案可以公开的验证，不是对所有规则集的覆盖保证。

另外，策略名也不必照抄 `PROXY`。应选择你自己配置里已经存在、并且确认可用的策略。

## 只把正在使用的配置再选一次，能当修复吗？

在本案例中不能。14:06 只使用同一份 `default.conf` 且不改规则，14:09 和 14:19 的 TLS 仍然失败。这不是有效修复。


---

<a id="chapter-6"></a>

## 隐私要求与脱敏说明模板

上传 GitHub 或转发给他人之前，请先按本页检查。本包在整理过程中不应读取或复制凭据、cookie、`auth.json`、会话库、真实配置文件或数据库；公开文件里也不应出现这些内容。

## 公开文件禁止写入的内容

- 电子邮箱、电话、真实姓名、可指向个人的用户名
- 机器名、主机名、设备 ID / machineId
- 账号 ID、工作空间 ID、组织 ID、会话 ID，以及带有这些 ID 的完整链接
- 节点密钥、订阅链接、服务商域名、配置订阅 URL、代理服务器公网 IP、节点或出口国家 / 地区名
- 聊天原文、私有路径、原始截图（截图里很容易带上节点名、邮箱或主机名）
- 进程 PID、绝对个人目录，以及完整的本地配置或数据库副本
- 内部任务编号或未公开的跟踪号

## 可以保留、但需要解释的信息

- 本机回环地址 `127.0.0.1` 与本地端口 `1082`（不是远端公网地址）
- 地址段 `198.18.0.0/15` 以及落入该段的观测地址（隧道 / fake-IP 机制地址，不是公网出口）
- 通用接口类别（例如 macOS 上的 `utun`）
- 已核对的应用版本、build 号，以及按应用分别记录的代码签名身份与 Gatekeeper 状态。不要把某一应用的校验结果套到另一应用；也不要把 Gatekeeper accepted 自动写成完整签名验证通过
- 界面文案、规则类型与策略名的**示例**、等待时延和客户端错误码
- 通用配置文件名（如本案中的 `default.conf`），不要同时贴出文件内容

节点只用「节点 A / 节点 B」泛称。

## 可复制模板（供遇到相似情况、需要向官方或社区说明时使用）

下面只使用本案例已经核对、且可公开的事实。若需按你的环境增删句子，请只保留与你本机一致的内容，不要补造没有发生过的步骤或「已命中」日志。向官方了解产品状态时，请从 [https://help.openai.com/](https://help.openai.com/) 进入，不要使用可能失效的旧文章链接。

```text
Summary:
On macOS, Muse recovered after changing the system HTTP proxy type from HTTP to None (TUN). Codex desktop remained broken until a local domain rule DOMAIN-SUFFIX,chatgpt.com,<EXISTING_POLICY> was added. The policy name used in this case was an already-existing policy shown as PROXY; others should choose a policy that already exists and works in their own config.

Environment:
- OS: macOS
- Shadowrocket (iOS-compatible build) 2.2.93
- Muse 12.1 (build 1114886626); code signing identity: Meta; Gatekeeper accepted (notarized)
- Codex desktop 26.1117.41037 (build 29803); code signing identity: OpenAI; Gatekeeper accepted. A deeper signature verify was not completed (Permission denied), so this should not be described as a full signature verification
- After the Muse fix: proxy type None (TUN); effective system HTTP/HTTPS proxy was no longer present; VPN/TUN remained enabled
- I did not reconnect the VPN or switch to another config when adding the rule

Muse (resolved in this case):
- Failure was consistent with the effective system HTTP/HTTPS proxy at 127.0.0.1:1082 resetting connections before TLS
- Changing only the proxy type from HTTP to None (UI shows TUN) restored Muse (HTTPS 200). I did not log out, clear cache, or reinstall.

Codex symptoms before the rule change:
- "Couldn’t load workspace settings"
- After a real app update (the main process was actually replaced), a new chat stayed on "Starting chat"
- "Reconnecting" reached 5/5
- "workspace routing discovery timed out"
- account/read had earlier waited about 10881 ms and 25869 ms, then failed with -32603
- No 401/403 or explicit account-disabled wording was found in the logs reviewed; some status 500 fields were locally wrapped and should not be read as a confirmed upstream HTTP 500

Network observations (redacted):
- Before the new rule, request logs showed chatgpt.com:443 hitting DOMAIN,chatgpt.com,DIRECT, and ab.chatgpt.com hitting DOMAIN-SUFFIX,chatgpt.com,DIRECT
- The in-use config editor search for chatgpt.com returned no results at that time. I am not attributing this discrepancy to cache or a software bug, because the mechanism was not verified
- Unauthenticated curl to the public host chatgpt.com failed at 14:19:54-14:19:59 (timezone Asia/Shanghai) with a TLS timeout of about 5 seconds, HTTP 000, curl exit code 28. HTTP 000 means no HTTP response was received; it is not a server status code
- After saving DOMAIN-SUFFIX,chatgpt.com,<EXISTING_POLICY> at 14:24, the same public target succeeded at 14:25:00-14:25:02: certificate verified, HTTP 200, curl exit code 0, TLS about 1.03s, total about 1.57s
- I do not have a final request-log screenshot showing the new rule being hit, so I am not claiming the new rule's priority was verified
- Codex account/read and getAuthStatus later returned in about 2-8 ms with errorCode=null; this may be cached and is not used alone as proof of success
- The app issue was confirmed resolved by the user at 14:26

What did not fix Codex in this case:
- The Muse proxy-type change alone
- Closing the window or bringing the app back did not verify a real quit; a later actual update replaced the process, and the error remained
- Clicking another VPN node while keeping None/TUN (no verified improvement; the header name and the selected-node indicator were also inconsistent, so actual egress was not verified)
- Re-selecting the same in-use config without changing rules (TLS still failed afterward)
```

中文备用表述：

```text
摘要：
macOS 上，Muse 在把系统 HTTP 代理类型从 HTTP 改为 None（TUN）后恢复。Codex 桌面应用仍失败，直到本机新增规则 DOMAIN-SUFFIX,chatgpt.com,<已有可用策略>。本案策略名使用了编辑页预填的已有策略 PROXY；其他人应选择自己配置中已存在且可用的策略。

环境：
- 系统：macOS
- Shadowrocket（iOS 兼容）2.2.93
- Muse 12.1（build 1114886626）；代码签名身份为 Meta，Gatekeeper 接受（notarized）
- Codex 26.1117.41037（build 29803）；代码签名身份为 OpenAI，Gatekeeper 为 accepted。深度签名校验遇到 Permission denied，不能表述为完整签名验证通过
- Muse 修复后：代理类型为 None（TUN），有效系统 HTTP/HTTPS 代理已不存在，VPN / TUN 仍启用
- 新增规则时没有重连 VPN，也没有切换配置

Muse（本案已恢复）：
- 故障与有效系统代理 127.0.0.1:1082 在 TLS 前重置连接相符
- 只把代理类型改为 None（界面明示 TUN）后，Muse 出现 HTTPS 200 并恢复；未退出登录、清缓存或重装

Codex 在改规则前的症状：
- Couldn’t load workspace settings；随后对话停在 Starting chat；Reconnecting 至 5/5；workspace routing discovery timed out。关窗口或重新唤起并未验证应用真正退出；后来实际更新更换了进程，仍报错
- 更早的 account/read 约 10881 ms、25869 ms 后失败，错误码 -32603
- 已查日志未见 401 / 403 或明确封禁文案；部分 status 500 为本地包装，不能写成服务端 HTTP 500

网络观察（已脱敏）：
- 新增规则前，请求日志出现 chatgpt.com:443 命中 DOMAIN,chatgpt.com,DIRECT，以及 ab.chatgpt.com 命中 DOMAIN-SUFFIX,chatgpt.com,DIRECT
- 当时正在使用的配置编辑页搜索 chatgpt.com 无结果。这一不一致的机制未证实，因此不归因于缓存或软件 bug
- 对公共目标 chatgpt.com 的无认证 curl，在 14:19:54–14:19:59（北京时间）TLS 超时约 5 秒，HTTP 000，curl 28。000 不是服务器状态码
- 14:24 保存 DOMAIN-SUFFIX,chatgpt.com,<已有可用策略> 后，同一目标在 14:25:00–14:25:02 证书校验通过，HTTP 200，curl 0；TLS 约 1.03 秒，总耗时约 1.57 秒
- 没有新规则被命中的最终请求日志截图，因此不声称已核验新规则的优先级
- 之后 Codex 的 account/read、getAuthStatus 多次约 2–8 ms 且 errorCode=null，可能是缓存，不能单独作为成功证据
- 用户于 14:26 确认问题已解决

本案中没有修好 Codex 的尝试：
- 只做 Muse 的代理类型变更
- 关窗口或重新唤起，未验证应用真正退出；随后实际更新更换了进程，仍报错
- 保留 None / TUN 时点选更换节点（无已验证改善；顶部名称与选中标记曾不一致，实际出口未证实）
- 只改用同一份正在使用的配置、不改规则（之后 TLS 仍失败）
```

