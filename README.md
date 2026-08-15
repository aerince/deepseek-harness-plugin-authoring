# DeepSeek Harness Plugin Authoring Skill

用于创建、调试、打包、安装和发布 DeepSeek Harness（DSH）插件的 Agent Skill。

它本身不是 DSH 插件，而是一套面向 OpenCode、Claude Code 等 Agent Skill 运行时的开发流程与模板。

## 能做什么

- 创建最小 Cordis/DSH host 插件
- 注册模型工具、配置、服务和事件
- 生成 `dsh.bundle` 与 `cordis.patch.yml`
- 诊断“安装成功但未激活”等常见问题
- 按当前官方约束构建 `dsh.client` Web 插件
- 验证本地、GitHub source 和 profile 安装路径
- 创建 GitHub 仓库、SemVer Release 和 `dsh-plugin` topic
- 准备 [tjsdyy/dshplugin](https://github.com/tjsdyy/dshplugin#development) 市场提交材料

## 安装

OpenCode 全局 Skill：

```sh
git clone https://github.com/aerince/deepseek-harness-plugin-authoring.git \
  ~/.config/opencode/skills/deepseek-harness-plugin-authoring
```

OpenCode 项目 Skill：

```sh
git clone https://github.com/aerince/deepseek-harness-plugin-authoring.git \
  .opencode/skills/deepseek-harness-plugin-authoring
```

Claude Code：

```sh
git clone https://github.com/aerince/deepseek-harness-plugin-authoring.git \
  ~/.claude/skills/deepseek-harness-plugin-authoring
```

安装后重启对应客户端，使新 Skill 被重新扫描。

## 使用示例

```text
帮我写一个 DeepSeek Harness 工具插件，并给出可测试的 profile 安装命令。
```

```text
检查这个 DSH 插件为什么 plugin add 成功后没有出现在 dump-config。
```

```text
把这个 Cordis 插件整理成可从 GitHub 安装的 dsh.bundle，并发布 v0.1.0。
```

## 设计原则

- 官方 `deepseek-ai/deepseek-harness` 文档和源码优先
- 默认生成无构建链、可直接 GitHub 安装的 JavaScript bundle
- TypeScript、Service、事件和 Web client 仅在需求需要时启用
- 不使用已移除的 `.dsh-plugin` 格式
- 不采用未经官方确认的 `dsh.skills`、`dsh.mcpServers` 字段
- 发布前留下真实可执行的安装与验证命令

## 版本基线

当前参考已核对到：

- DeepSeek Harness commit `47f943859bef60e4160492346772ded9b24f765a`
- source version `0.1.0-rc.5`
- 核对日期 `2026-08-15`

DSH 仍处于 developer preview。Skill 会要求在关键接口上先检查官方当前版本。

## 仓库结构

```text
.
├── SKILL.md
├── references/
│   ├── official-contract.md
│   ├── templates.md
│   ├── web-client.md
│   └── github-publishing.md
├── evals/evals.json
├── README.md
└── LICENSE
```

## License

[MIT](LICENSE)
