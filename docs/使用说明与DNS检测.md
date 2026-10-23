# Shadowrocket 配置包

制作与核验日期：2026-10-24（北京时间）。按“已有节点、国内直连、国外代理”制作。文件不含节点、订阅、账号或密码。

## 先用哪一个

日常使用 `01-Daily-CN-Direct.conf`。国内常用服务、规则列出的苹果与微软国内 CDN 直连；其余请求使用首页选中的可用节点。规则匹配取决于域名集合和 IP 数据库，不能保证每个国内 App 的所有请求都自动直连。

`02-Full-Proxy-Compare.conf` 用于对照检查：普通互联网目的地全部代理，保留局域网 IP 直连，节点域名仍通过国内加密 DNS 进行启动解析。国内游戏可能因此增加延迟。它不是“关闭 VPN 后整机断网”的系统防火墙。

两个配置都需要支持直连专用 DNS 的 Shadowrocket，建议 **2.2.92 或更新版本**。本次 [App Store 正式版更新说明](https://apps.apple.com/us/app/shadowrocket/id932747118)已列出 dedicated DNS routing for direct domains。旧版可能忽略新参数，不应套用本文的路径预期。

## 导入与启用

1. 先在 Shadowrocket 保存当前配置副本，并记下当前启用模块，方便恢复。
2. 将 `.conf` 文件存到 iCloud Drive / iPhone“文件”，通过文件分享“拷贝到 Shadowrocket”，或在 Shadowrocket 配置页面使用 iCloud Drive 导入。不同版本入口名称可能略有差异。
3. 在“配置 → 本地文件”选中 `01-Daily-CN-Direct.conf`，确认出现选中标记；首页“全局路由”选 **配置**，首页选你已有的可用节点，然后连接。
4. 首次使用需下载日常配置引用的三份远程规则。确认下载成功；GitHub 直连受阻时，先选可用节点，把全局路由临时改为“代理”下载，完成后切回“配置”。不要在远程规则下载失败时直接判断国内分流效果。
5. 先关闭旧配置遗留的规则、DNS、重写类模块，完成基础验证后再逐个启用本包模块。模块优先级高于主配置，旧模块可能改变 DNS 或直连策略。

**不需要安装 CA 根证书，也不需要开启 HTTPS 解密。** 本包没有 MITM、远程 JavaScript、会员解锁或应用响应修改。

### 日常配置下下载超时的排查

如果同一节点在“配置”模式下载模块超时，而全局路由改成“代理”后能下载，先保持节点不变，在代理模式执行日常配置的“使用配置 / 编译配置”，确认“规则集 URL”里三份远程资源全部下载成功（✅），再把全局路由改回“配置”重试同一模块。

2026-10-24 用户已确认：在代理模式重新使用 / 编译配置后，切回配置模式也能正常下载模块。此次恢复没有修改 DNS 或分流规则；结果支持优先检查加载 / 编译状态，具体底层原因仍未通过日志确认。若在其他网络再次发生且重编译无效，可提供 Shadowrocket 版本及 `raw.githubusercontent.com` 对应的代理 / DNS 日志，不要提交含节点凭据的完整配置。

### 三个可选模块

在“配置 → 模块 → 新建模块”中，粘贴对应 `.sgmodule` 文件全文并保存；若当前版本支持文件导入，也可直接导入。这里提供的是本地文件，没有伪造公网下载链接。

| 文件 | 作用 | 初次使用建议 |
|---|---|---|
| `01-Ads-Lite.sgmodule` | 自行编写的 10 条广告/统计域名规则，无脚本和证书 | 基础联网正常后可试开；部分 App 的广告奖励、统计或内容加载可能受影响 |
| `02-Proxy-QUIC-Block.sgmodule` | 用 `block-quic=all-proxy` 让代理连接的 QUIC 尝试回落到 TCP；国内 DIRECT UDP 不受此项限制 | 默认关闭；代理视频/网页卡顿时再试，QUIC 专用应用可能无法使用 |
| `03-Block-HTTPDNS-Advanced.sgmodule` | 引用 blackmatrix7 的已知 HTTPDNS 地址阻断规则 | **默认关闭**；微信、网易、阿里等国内业务可能变慢或失效，不能当作全面防 DNS 泄露开关 |

前两个模块为本次独立编写；第三个模块只引用公开规则。模块不会因为放进文件夹就自动启用。轻量广告模块不能去除与正常内容共用域名的广告，也不保证去除 YouTube 视频广告或所有开屏广告。

## DNS 实际怎样走

| 请求类别 | 配置预期 |
|---|---|
| 命中国内直连域名规则 | 正常情况下通过 HTTPS 直连 AliDNS：`223.5.5.5` / `223.6.6.6`，用于国内 CDN 调度；失败时可能转到代理 DoH 备用解析 |
| 其他流量需要 Shadowrocket 在本机发起解析 | 通过所选节点访问 Cloudflare / Google DoH；备用也使用代理 DoH |
| 代理协议直接把目标域名交给服务器 | 由节点服务器解析，实际解析器由节点提供方决定；不能保证检测页只显示 Cloudflare / Google |
| 代理节点自身域名的启动解析 | 明确使用国内 AliDNS 的 IP 形式 DoH；这是建立代理前有意保留的加密直连，AliDNS 可看到节点域名 |
| 应用自带 HTTPDNS、DoH、DoT | 作为应用的网络连接匹配分流规则，可能走 DIRECT 或 PROXY；不会全部被端口 53 劫持接管 |

`dns-direct-system=false` 避免强制直连域名使用系统 DNS。备用 DNS 明确写成加密上游，没有配置成 `system`、空值或普通 UDP DNS。配置不启用 `dns-direct-fallback-proxy`，但这不等于禁止所有解析回退：直连专用 DNS 失败仍可能按 `fallback-dns-server` 使用代理 DoH，国内查询也可能因此经代理备用解析。备用也失败时会解析失败，具体以手机日志为准。

`hijack-dns` 仅列出 8 个已知公共 DNS 端点，不是全端口、全应用捕获。没有写入支持情况不明确的 `dns-fallback-system` 或 `include-all-networks` 文本参数。

`udp-policy-not-supported-behaviour=REJECT` 避免节点不支持 UDP 时自动转为直连；正常命中 DIRECT 的国内游戏依旧可以直连 UDP。

`GEOIP,CN,DIRECT,no-resolve` 不会仅为判定未知域名是否属于国内而强制发起额外 DNS 查询，因此国内域名集合遗漏的站点可能走代理。遇到具体 App 变慢，先查看实际请求域名，再添加准确的直连规则；不建议把全部腾讯或全部 UDP 无条件放到规则最上面。

## 手机会不会 DNS 泄露

**会有可能。这份配置降低常见风险，不能仅凭文件证明你的 iPhone 没有泄露。**

需要区分三件事：DNS 是否明文、解析连接是否经过代理、解析器是否能看到真实来源。直连国内 DoH 加密了查询内容，但 AliDNS 仍能看到来自你的连接和所查询的域名；这是日常分流的预期。国外查询若意外送给本地运营商 DNS，或者应代理的解析连接走了直连，才不符合本方案的预期。[DoH 的隐私边界见 RFC 8484 第 8 节](https://datatracker.ietf.org/doc/html/rfc8484#section-8)。

还需要检查手机设置与连接状态：

- Shadowrocket“设置 → 隧道 → 包括所有网络”可用于提高隧道接管范围；先记录当前状态，开启后验证。此开关本身不等于把日常配置里的所有 DIRECT 规则改为代理。AirPlay、CarPlay、通知等如有异常，应结合其“包括本地网络 / APNs / 蜂窝服务”等具体选项逐项排查。
- 可开启按需求连接，减少忘记连接的情况；它不等于 MDM 受监管设备的 Always On VPN，也不保证你手动关闭 Shadowrocket 后继续阻断整机联网。[Apple 对 VPN 路由与系统例外的说明](https://developer.apple.com/documentation/networkextension/routing-your-vpn-network-traffic)。
- 本包 `ipv6=false` 只是减少 Shadowrocket 的 IPv6 变量，不等于关闭 iPhone 整机 IPv6，也不保证节点连接不使用 IPv6。IPv6-only / NAT64 网络若不可用，应改为支持 IPv6 的设置后重新测试，不能为了检测页面“没有 IPv6”而忽略连通性。
- 应用自己的解析、其他 DNS 描述文件、iCloud Private Relay、网络切换和 VPN 未连接阶段，都需要单独观察。若启用了其他隐私/加密 DNS 功能，记录状态，必要时逐项对照，避免把不同功能的结果混在一起。

## 手机上怎么验证

1. 模块先全部关闭，连接日常配置，分别使用 Wi-Fi 和蜂窝网络测试。
2. 打开 [DNSLeakTest](https://www.dnsleaktest.com/) 运行 Extended test，再用 [BrowserLeaks DNS](https://browserleaks.com/dns) 交叉检查。同时看公网 IPv4、IPv6 和 WebRTC。网页看到的解析器可能属于节点提供方；递归解析器 IP 与代理出口 IP 不必完全相同。
3. 在 Shadowrocket“数据 → DNS 日志 / 代理日志”查看测试随机域名、上游和最终策略。部分版本日志可能不展示所有细节，日志不足时需在可信路由器上辅助抓包，不能把“没看到记录”当作无泄露。
4. 访问百度、淘宝、微信或你实际玩的国内游戏，确认相应业务请求命中 DIRECT；再访问 GitHub、Google 等，确认相应请求命中 PROXY。DNS 检测站不会替你验证每个 App。
5. 切换全代理对照配置，重复测试。随后测试 Wi-Fi 切蜂窝、锁屏唤醒、节点不可达与重连时的情况。节点不可达和手动关掉 VPN 是不同场景，需要分别记录结果。
6. 每次只启用一个可选模块后复测；国内游戏或 App 异常时优先关闭 HTTPDNS 模块，保留日志定位具体域名。

判断标准：国外测试随机域名不应意外由本地运营商 DNS 解析；日常模式出现预期的国内 DoH 查询并不自动等于泄露。**浏览器检测通过只说明该次浏览器查询的观察结果，不能证明所有 App、所有网络和断线时刻都无泄露。** 本包没有给检测网站添加专门的代理例外规则。

## 来源、维护与更新

| 作者 / 项目 | 本包用途 | 核验结果 |
|---|---|---|
| [SukkaW/Surge](https://github.com/SukkaW/Surge)，生成仓库 [SukkaLab/ruleset.skk.moe](https://github.com/SukkaLab/ruleset.skk.moe) | 日常国内域名、苹果国内 CDN、微软国内 CDN | 分别为 873、160、53 条，内容日期 2026-10-20 / 08-26 / 10-15；AGPL-3.0 |
| [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket) | 参数、模块格式与操作方法参考 | 非官方社区手册；配置更新 2026-10-15，没有把其完整配置重新分发 |
| [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script) | 可选 HTTPDNS 规则 | 63 条，文件内容日期 2025-06-06；GPL-2.0。仓库近期有构建不代表此文件每天更新 |
| [Johnshall/Shadowrocket-ADBlock-Rules-Forever](https://github.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever) | 现成配置的备选阅读来源 | 本次核到 2026-10-23 北京时间构建，CC BY-SA 4.0；未直接套用其重写/MITM配置 |

Sukka 规则固定到提交 `6c5bee5b8768fe092b85572e8f2c0ec0a900d44a`；blackmatrix7 固定到 `00d82f6c6c374abb947fabc9c1b69f2a22e71dbc`。远程内容以引用方式使用，没有把第三方数据改名打包。

固定提交便于复核，但不会跟随最新分支变化；在 Shadowrocket 点击“更新规则”仍只会重新下载这一版本。想跟随维护者更新，可先备份，再把 URL 中的对应 40 位提交号改成 `master`，下载后检查变更和实际业务；这会允许后续远程内容变化。也可以定期核验新提交后固定到新的提交号。各项目许可见仓库 LICENSE。

DNS 端点及语法依据：[Cloudflare DoH](https://developers.cloudflare.com/1.1.1.1/encryption/dns-over-https/make-api-requests/)、[Google DoH](https://developers.google.com/speed/public-dns/docs/doh/)、[阿里公共 DNS API 文档](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/download%2Fpdf%2F171662%2FAPI_Reference_intl_en-US.pdf)、[Shadowrocket 官方频道关于 proxy-dns-server 的说明](https://t.me/s/ShadowrocketNews/934)、[LOWERTOP 配置参考](https://github.com/LOWERTOP/Shadowrocket/blob/main/lazy.conf)。

## 已完成的核验与未完成的验证

已读取实际引用文件，核对存在性、格式、固定提交和许可证；已检查本包配置结构、DNS 设置、策略和不含脚本/证书。两个 AliDNS IP 形式 DoH 在当前 Windows 环境经过正常 TLS 校验得到 HTTP 200、application/dns-message、DNS RCODE 0 和有效 A 记录响应。

**没有在你的 iPhone 上导入、运行或抓包，也没有验证你的节点协议和所有国内游戏。** 最终需要按上面的手机步骤验证。出现问题时可提供 iOS / Shadowrocket 版本、出问题的 App 和脱敏日志，不要发送订阅链接、节点密码或证书私钥。

