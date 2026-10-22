# DeepSeek Harness Plugin Authoring Skill

面向 OpenCode、Claude Code 等 Agent Skill 运行时的 DeepSeek Harness（DSH）插件开发技能：从最小 Cordis host plugin，到 GitHub source 安装、验证、Release 和 marketplace 提交。

**本仓库不是 DSH 插件。** 它没有 `package.json`、`dsh.bundle` 或 `cordis.patch.yml`，因此不能、也不应该用 `dsh plugin --profile ... add` 安装。

## 能做什么

- 创建最小 Cordis/DSH host 插件
- 注册模型工具、配置、服务和事件
- 生成 `dsh.bundle` 与 `cordis.patch.yml`
- 诊断“安装成功但未激活”等常见问题
- 按当前官方约束构建 `dsh.client` Web 插件
- 验证本地、GitHub source 和 profile 安装路径
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

Windows PowerShell 可将目标目录改为 `$env:USERPROFILE\.config\opencode\skills\deepseek-harness-plugin-authoring` 或 `$env:USERPROFILE\.claude\skills\deepseek-harness-plugin-authoring`。安装后重启对应客户端，使 Skill 重新扫描。

## 验证安装

确认目标目录中存在 `SKILL.md`、`references/official-contract.md` 和 `references/github-publishing.md`，然后重启客户端并发出类似请求：

```text
帮我写一个 DeepSeek Harness 工具插件，并给出可测试的 profile 安装命令。
```

本 Skill 的安装验证是“客户端能发现并调用 Skill”，不是 `dsh --profile ... --dump-config`。后者只用于验证由本 Skill 创建的 DSH 插件。

## DSH 官方安装对照

| 项目 | 正确方式 |
|---|---|
| 本仓库类型 | Agent Skill，不是 DSH bundle |
| 本仓库安装 | `git clone` 到客户端的 Skill 目录 |
| `dsh plugin --profile ... add` | 不适用；不要把本仓库当 DSH 插件安装 |
| 本 Skill 生成的插件 | 应包含普通 package entry、`dsh.bundle.patch`、已验证的 `engines.dsh` 和 `cordis.patch.yml` |
| 生成插件的验证 | `dsh --profile <profile> --dump-config` |

官方当前核对基线：`deepseek-ai/deepseek-harness` commit `687f282e8625c6b8d11ff58211a90524be2e4317`，release tag `dsh-v0.2.1-rc.2`（2026-10-22 核对）。DSH 仍处于 developer preview，Skill 在关键接口上要求先检查官方当前版本。

官方参考：<https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/publish.zh.md>、<https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/src/plugin.ts>。

## 设计原则

- 官方 `deepseek-ai/deepseek-harness` 文档和源码优先
- 默认生成无构建链、可直接 GitHub 安装的 JavaScript bundle
- TypeScript、Service、事件和 Web client 仅在需求需要时启用
- 不使用已移除的 `.dsh-plugin` 格式
- 不采用未经官方确认的 `dsh.skills`、`dsh.mcpServers` 字段
- 发布前留下真实可执行的安装与验证命令

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
