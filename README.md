# Mihomo 规则集

本仓库的规则集均为逐行 TXT，provider 必须设置 `format: text`，文件中不加 YAML 的 `payload:` 或列表前缀。出口策略写在主配置的 `RULE-SET` 中。

## 文件与 behavior

| 文件 | behavior | 内容及注意事项 |
| --- | --- | --- |
| `direct.txt` | `domain` | 3 条后缀、1 条精确域名；`dns.weixin.qq.com.cn` 仅精确匹配 |
| `default_proxy.txt` | `classical` | 包含 `DOMAIN-KEYWORD`，保留关键词及后缀匹配 |
| `default_reject.txt` | `domain` | 11 条后缀；`+.example.com` 匹配根域名及其子域名 |
| `direct_ip.txt` | `ipcidr` | 纯 CIDR；匹配时保留默认的域名解析行为 |
| `github_cidr.txt` | `classical` | IPv4/IPv6 CIDR；保留每条规则的 `no-resolve` |
| `ip-proxy.txt` | `classical` | CIDR；保留每条规则的 `no-resolve` |

`github_cidr.txt` 和 `ip-proxy.txt` 保留 classical，以便现有 `RULE-SET` 引用继续继承文件内的 `no-resolve`。直接删去这些参数并转换成 ipcidr，会改变域名解析行为。

## 配置引用

[examples/rule-providers.yaml](examples/rule-providers.yaml) 提供全部 6 个 provider 的声明。合并到现有主配置时保留当前 provider 名称、策略组和规则顺序；示例使用的名称可以按原配置调整。

升级时需同步更新文件内容与 provider 声明：`direct.txt`、`default_reject.txt` 使用 `behavior: domain`，`direct_ip.txt` 使用 `behavior: ipcidr`。三者均使用 `format: text`；旧的 classical 声明不能继续读取转换后的内容。HTTP 缓存需要重新下载，不能继续使用旧内容。

下面是对应的独立 `rules` 引用示例。应在原规则所在位置更新名称，保留原出口与先后顺序；此列表不定义完整主配置的顺序。

```yaml
rules:
  - RULE-SET,my_default_direct,DIRECT
  - RULE-SET,my_default_proxy,Proxy
  - RULE-SET,my_default_reject,REJECT
  - RULE-SET,direct_ip,DIRECT
  - RULE-SET,github_cidr,Proxy
  - RULE-SET,ip_proxy,Proxy
```

`Proxy` 必须对应现有策略组。不要把 `DIRECT`、`Proxy`、`REJECT` 等策略名写进规则集；`no-resolve` 是有效规则参数，应保留。

DNS 的 `rule-set:` 引用可以使用 `my_default_direct`、`my_default_reject` 这类 domain provider。`my_default_proxy` 需要保留 classical，供 DNS 策略引用时仅域名规则参与匹配，相关 Classical 提示是预期行为。不要将纯 IP provider 加入 `dns.nameserver-policy`，也不要为消除提示而删除关键词规则。

例如，已有 DNS 策略键可继续使用 `rule-set:my_default_direct`、`rule-set:my_default_proxy`；保留各自原有的 DNS 上游及策略顺序。修改 provider 名称时，必须同步修改 `rules` 和 DNS 中的所有引用。

当前主分支未包含实际主配置，因此此处只能提供配套声明与引用说明，无法确认外部主配置的规则顺序、策略组、DNS 上游或 `telegram` 等外部 provider。

## CIDR 范围

| 原写法 | 规范写法 | 实际匹配范围 |
| --- | --- | --- |
| `101.33.69.0/22` | `101.33.68.0/22` | `101.33.68.0–101.33.71.255` |
| `192.3.86.117/24` | `192.3.86.0/24` | `192.3.86.0–192.3.86.255` |
| `20.57.32.77/24` | `20.57.32.0/24` | `20.57.32.0–20.57.32.255` |

以上仅规范网络地址，保留原前缀长度和覆盖范围。

历史日志中的 `IP-CIDR.101.43.105.233/22` 使用了错误分隔符。classical 中的正确写法为 `IP-CIDR,101.43.104.0/22`；ipcidr 中则为 `101.43.104.0/22`，覆盖 `101.43.104.0–101.43.107.255` 共 1024 个地址。若只需单个地址，应明确使用 `/32`。该条规则已由后续历史提交删除，本次不恢复。

## 官方格式参考

- [规则集合声明](https://wiki.metacubex.one/config/rule-providers/)
- [规则集合内容](https://wiki.metacubex.one/config/rule-providers/content/)
- [路由规则及 no-resolve](https://wiki.metacubex.one/config/rules/)
- [DNS 配置](https://wiki.metacubex.one/config/dns/)
