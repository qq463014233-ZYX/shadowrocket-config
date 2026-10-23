# Shadowrocket 国内直连、国外代理配置

制作日期：2026-10-24（北京时间）。适用于已经有可用节点的 iPhone 用户；建议 Shadowrocket 2.2.92 或更新版本。配置和模块不包含节点、订阅、账号、密码或证书，不需要安装根证书或开启 HTTPS 解密。

日常配置将已匹配的国内服务和部分国内 CDN 直连，其他目的地使用 Shadowrocket 首页选择的节点。规则不可能覆盖每个国内 App 的全部请求，遇到异常请查看实际命中的策略。

## 手机导入

1. 先备份当前配置，记下已启用的模块。
2. 复制下方“日常配置”的完整 URL，在 Shadowrocket 的“配置”页点右上角“＋”，粘贴 URL 并下载；选中下载后的本地配置。
3. 首页“全局路由”选择“配置”，首页选择你已有的可用节点并连接。
4. 确认日常配置引用的远程规则下载成功。GitHub 连接失败时，可先使用原有可用配置和节点下载，再切换到本配置。
5. 初次测试时关闭三个可选模块，以及旧配置遗留的 DNS、规则和重写模块。完成基础联网与 DNS 测试后，每次只启用一个模块。

模块：复制对应 `.sgmodule` URL，在“配置 → 模块”中使用新增/添加入口下载，初次导入后保持关闭。若当前版本没有 URL 导入入口，可打开 raw 链接复制全文，在“新建模块”中粘贴保存。界面名称可能随版本变化。

不要用浏览器地址栏显示的 GitHub 文件网页代替以下 raw 文件 URL。若 Safari 只是显示文本，把链接复制进 Shadowrocket 即可。

## 日常配置下载超时，但全局代理能下载

这说明同一节点能够访问下载地址，应先检查配置模式的资源加载和实际路由；不能仅凭这个现象确定是 DNS 泄露或模块内容错误。

1. 保持原节点不变，首页“全局路由”临时选择“代理”。
2. 在配置页点击日常配置，执行“使用配置 / 编译配置”；在“规则集 URL”中确认三份远程规则均下载成功（✅）。不同版本入口可能略有差异。
3. 全局路由改回“配置”，重新下载同一个模块，比较结果。代理模式下载成功不代表配置模式已经恢复，必须在切回后复测。
4. 若仍超时，记录 Shadowrocket 版本，并在代理 / DNS 日志中查看 `raw.githubusercontent.com` 的策略与错误阶段。反馈时不要发送节点密码、订阅地址或证书私钥。

2026-10-24 已收到手机复测反馈：同一日常配置、同一节点，在代理模式重新“使用配置 / 编译配置”后，切回配置模式已能正常下载模块。恢复过程没有修改 DNS 或分流规则；这更符合配置加载 / 编译状态问题，具体底层原因尚未通过日志确认。

## 5 个导入直链

### 日常配置（先用这个）

[01-Daily-CN-Direct.conf](https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/profiles/01-Daily-CN-Direct.conf)

```text
https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/profiles/01-Daily-CN-Direct.conf
```

### 全代理对照配置（测试用）

[02-Full-Proxy-Compare.conf](https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/profiles/02-Full-Proxy-Compare.conf)

```text
https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/profiles/02-Full-Proxy-Compare.conf
```

普通互联网目的地走代理，保留局域网 IP 直连。用于对照 DNS 和分流结果，国内游戏可能增加延迟；不等于系统断网开关。

### 轻量广告模块（可选）

[01-Ads-Lite.sgmodule](https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/modules/01-Ads-Lite.sgmodule)

```text
https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/modules/01-Ads-Lite.sgmodule
```

10 条广告/统计域名规则。基础联网正常后再试开；不保证去除同域广告、视频广告或所有开屏广告，可能影响广告奖励和内容加载。

### 代理 QUIC 调整模块（默认关闭）

[02-Proxy-QUIC-Block.sgmodule](https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/modules/02-Proxy-QUIC-Block.sgmodule)

```text
https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/modules/02-Proxy-QUIC-Block.sgmodule
```

代理视频/网页不稳定时再尝试，让代理 QUIC 连接回落到 TCP；国内 DIRECT UDP 不受此项限制，QUIC 专用应用可能无法使用。

### 已知 HTTPDNS 阻断模块（高级选项，默认关闭）

[03-Block-HTTPDNS-Advanced.sgmodule](https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/modules/03-Block-HTTPDNS-Advanced.sgmodule)

```text
https://raw.githubusercontent.com/qq463014233-ZYX/shadowrocket-config/main/modules/03-Block-HTTPDNS-Advanced.sgmodule
```

可能使国内 App 和游戏变慢或失效，不能接管所有应用自带的 DNS，也不是全面防 DNS 泄露开关。日常使用建议保持关闭。

## DNS 路径与测试

- 国内直连域名通常通过 AliDNS 的 HTTPS DNS 解析，用于国内 CDN 调度；失败时可能使用配置中的代理 DoH 备用解析。这种预期的国内查询不自动等于 DNS 泄露。
- 需要由 Shadowrocket 在本机解析的其他域名，通过所选节点访问 Cloudflare / Google DoH。部分代理协议直接把域名交给节点服务器解析，实际解析器可能属于节点提供方。
- 节点自身域名的启动解析明确使用国内 AliDNS DoH；AliDNS 能看到节点域名。加密 DNS 不等于解析器看不到查询，也不等于所有解析都经过代理。
- 应用自带 DoH/HTTPDNS、网络切换、IPv6、其他 DNS 描述文件及 VPN 未连接期间仍需手机实测。配置不能证明所有 App 和断线时刻都无泄露。

先关闭模块，连接日常配置，用 Safari 打开 [DNSLeakTest](https://www.dnsleaktest.com/) 运行 Extended test，再用 [BrowserLeaks DNS](https://browserleaks.com/dns) 交叉检查。测试网页公网 IP 应符合代理出口；解析器 IP 不必与代理出口相同。国外随机测试域名若意外由本地运营商 DNS 解析，应结合 Shadowrocket 的 DNS/代理日志排查。

分别在 Wi-Fi、蜂窝网络测试，随后测试网络切换和锁屏唤醒，再切换全代理对照配置复测。浏览器测试通过只代表该次浏览器查询的观察结果。

详细说明与测试步骤见 [使用说明与 DNS 检测](docs/使用说明与DNS检测.md)。该文档保留本地文件导入方法；本页提供公网 URL 导入方法。尚未在你的 iPhone 或节点上进行导入、运行或抓包验证。

## 来源和固定版本

| 项目 | 用途 | 规则固定版本 |
|---|---|---|
| [SukkaW/Surge](https://github.com/SukkaW/Surge) / [SukkaLab/ruleset.skk.moe](https://github.com/SukkaLab/ruleset.skk.moe) | 国内域名、苹果与微软国内 CDN 规则 | [`6c5bee5b8768fe092b85572e8f2c0ec0a900d44a`](https://github.com/SukkaLab/ruleset.skk.moe/tree/6c5bee5b8768fe092b85572e8f2c0ec0a900d44a)，AGPL-3.0 |
| [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script) | 可选 HTTPDNS 阻断规则 | [`00d82f6c6c374abb947fabc9c1b69f2a22e71dbc`](https://github.com/blackmatrix7/ios_rule_script/tree/00d82f6c6c374abb947fabc9c1b69f2a22e71dbc)，GPL-2.0 |
| [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket) | 社区参数、模块格式和操作方法参考 | 未重新分发其完整配置 |
| [Johnshall/Shadowrocket-ADBlock-Rules-Forever](https://github.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever) | 现成配置的备选阅读来源 | 未重新分发其配置 |

远程第三方规则仅以 URL 引用，未打包进入本仓库。固定提交便于复核，不会随上游分支自动更新；Shadowrocket 的“更新规则”只会重新下载该固定版本。需要更新时应先检查上游变更，再将配置 URL 固定到新提交并复测。

本仓库的 `main` 导入链接会取得本仓库对应文件的当前版本；“固定版本”指配置内部引用的上游规则版本。自行修改配置前请保留副本。

## 许可

本仓库独立编写的配置、模块封装和说明按 [MIT License](LICENSE) 提供。远程引用的第三方规则、以及链接到的其他项目继续适用其各自许可证；MIT 许可不覆盖这些第三方内容。
