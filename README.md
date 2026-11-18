# Shadowrocket 自定义规则

日常修改 `rule_set_*.list` 或源配置 `sr_top500_banlist_ad.conf`，不要直接编辑生成文件 `sr_top500_banlist_ad_merged.conf`。

## 本地生成

需要 Python 3.9 或更高版本，在仓库根目录运行：

```sh
python3 scripts/merge_rules.py
```

脚本原位置展开源配置引用的本仓库自定义 RULE-SET，补上策略，保留分组注释和匹配顺序；保留 DOMAIN-SET、第三方规则集和其他配置。生成配置的 update-url 指向生成文件自身。脚本不下载网络资源。

## 白名单配置

`sr_top500_banlist_ad.conf` 不改。白名单单独使用这套文件：

- 源配置 `sr_source.conf` 只保留域名集引用，不展开
- `scripts/merge_sr.py` 读取该源配置，生成最终文件 `sr_merged.conf`
- 生成时展开 `domain_set_reject.list`、`domain_set_direct.list`、`domain_set_proxy.list`，以及 `domain_keyword_proxy.list`。`surge2.txt` 仍按远程 `DOMAIN-SET` 保留

这三个域名集来自 Johnshall 的白名单和黑名单，方便以后对照更新：

```text
https://raw.githubusercontent.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever/refs/heads/release/sr_top500_whitelist_ad.conf
https://raw.githubusercontent.com/Johnshall/Shadowrocket-ADBlock-Rules-Forever/refs/heads/release/sr_top500_banlist_ad.conf
```

两份文件的当前快照 build time 都是 UTC 2026-11-17 23:02:46。更新时只从 `[Rule]` 抽出 `DOMAIN-SUFFIX`，写成 `.域名`。同一个域名只保留一条：先用白名单里先出现的策略，白名单没有的再按黑名单里先出现的策略补上。黑名单没有新的 Direct 或 Reject 域名，多出来的 Proxy 域名补进 `domain_set_proxy.list`。IP 规则留在 `sr_source.conf`。

`domain_keyword_proxy.list` 每行一个关键词，用来后续自行追加。生成时写成 `DOMAIN-KEYWORD,关键词,PROXY`，放在代理域名集之后、直连域名集之前。现在有 `amazon` 和 `aws`。

```sh
python3 scripts/merge_sr.py
```

该脚本不修改 `sr_source.conf`，也不改黑名单那套文件。`sr_source.conf` 的 `update-url` 指向 `sr_merged.conf`。

## GitHub 自动更新

将脚本、工作流和生成文件提交并推送到 main。之后修改源配置或自定义规则并推送 main，会自动运行 **Generate merged Shadowrocket config**。生成内容变化时，机器人只提交生成文件；无变化时跳过提交。

也可以进入 GitHub → Actions → Generate merged Shadowrocket config → Run workflow，选择 main 手动运行。

工作流使用 GITHUB_TOKEN 的 contents: write 权限，不需要配置个人 Token。如果仓库或组织限制 Actions 写权限，或 main 分支保护禁止机器人推送，需按仓库策略调整权限或改用 PR 流程。推送冲突时不会强推，工作流会失败；可在 main 更新后重新手动运行。

## 手机使用

推送成功后，在 Shadowrocket 导入并选用以下配置，使用配置分流模式：

```text
https://raw.githubusercontent.com/angelen10/shadowrocket-rules/refs/heads/main/sr_top500_banlist_ad_merged.conf
```

后续等待 Actions 成功再更新手机配置。自定义规则直接出现在配置正文里；第三方远程规则集仍按原方式加载。手机临时编辑不会同步回仓库，长期修改应写入源文件。
