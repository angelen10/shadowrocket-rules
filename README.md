# Shadowrocket 自定义规则

日常修改 `rule_set_*.list` 或源配置 `sr_top500_banlist_ad.conf`，不要直接编辑生成文件 `sr_top500_banlist_ad_merged.conf`。

## 本地生成

需要 Python 3.9 或更高版本，在仓库根目录运行：

```sh
python3 scripts/merge_rules.py
```

脚本原位置展开源配置引用的本仓库自定义 RULE-SET，补上策略，保留分组注释和匹配顺序；保留 DOMAIN-SET、第三方规则集和其他配置。生成配置的 update-url 指向生成文件自身。脚本不下载网络资源。

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
