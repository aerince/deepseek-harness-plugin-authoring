---
name: deepseek-harness-plugin-authoring
description: DeepSeek Harness（DSH）插件开发、调试、打包、安装与 GitHub 发布指南。Use whenever the user asks to create, modify, debug, review, install, or publish a DeepSeek Harness/Cordis plugin; mentions dsh.bundle, dsh.client, cordis.patch.yml, defineTool, `dsh plugin --profile ... add`, GitHub source installation, releases, or dshplugin marketplace submission. Re-check official sources because DSH is a developer preview.
license: MIT
compatibility: Requires network access for current-version checks and GitHub access for publishing. Generated plugins should target the currently installed DeepSeek Harness version.
metadata:
  author: aerince
  version: "1.0.0"
---

# DeepSeek Harness Plugin Authoring

Create the smallest working DSH plugin, prove its install path, and publish it when requested.

## Read First

Read [references/official-contract.md](references/official-contract.md) for every task. DSH is a developer preview, so compare the current official `master` commit with the reference baseline before relying on unstable interfaces.

Load only the extra reference needed by the request:

- Scaffolding or fixing package files: [references/templates.md](references/templates.md)
- Browser UI or `dsh.client`: [references/web-client.md](references/web-client.md)
- GitHub push, release, or marketplace submission: [references/github-publishing.md](references/github-publishing.md)

## Workflow

### 1. Inspect the actual target

If a repository exists, read only its `package.json`, plugin entry, `cordis.patch.yml`, build config, README install section, and relevant callers. Reuse its package manager and build pattern.

If no repository exists, infer a kebab-case package name from the requested capability and use the zero-build JavaScript bundle in `references/templates.md`. State the assumption; do not block on naming unless a collision matters.

### 2. Choose one plugin shape

Stop at the first shape that satisfies the request:

1. Plain host plugin: `apply(ctx)`.
2. Model tool: host plugin plus `ctx.tools.register(defineTool(...))`.
3. Configurable plugin: same-named TypeScript `Config` interface and Schemastery schema.
4. Service provider: class extending `Service` only when other plugins need a reusable service.
5. Event extension: typed Cordis event when loose coupling is useful.
6. Web client: host half plus `dsh.client` and `exports["./client"]`; use only when browser UI is explicitly required.

Do not add a framework, factory, registry, test suite, or build system unless the chosen shape needs it.

### 3. Implement lifecycle and dependencies correctly

- Prefer the named function form: `export function apply(ctx: Context)`.
- Declare required services in `export const inject = [...]`.
- Query optional services with `ctx.get()` at the use site.
- Registrations made through `ctx` are effect-scoped and clean up on unload.
- Wrap manually owned resources in `ctx.effect()` and return their cleanup function.
- Put deployment-adjustable values in the exported Schemastery config schema.
- Let invalid configuration fail loudly.

### 4. Make it installable only when distribution is requested

An installable bundle needs:

- a normal package entry;
- `cordis.patch.yml`;
- `package.json` field `dsh.bundle.patch` pointing to that patch;
- a patch row whose `name` resolves to the installed package.

Use zero-build JavaScript by default for GitHub source installation. For TypeScript, either ship built output or provide a self-contained `prepare`; explain that pnpm 10+ requires the installer to authorize that package under profile `allowBuilds`.

Never create the removed `.dsh-plugin` manifest or repository-plugin wrapper.

### 5. Run one proof for each changed boundary

Use the smallest commands available in the target environment:

```sh
# Package contents
pnpm pack

# Install into a disposable profile
dsh plugin --profile plugin-test add .
dsh --profile plugin-test --dump-config

# Remove after the check
dsh plugin --profile plugin-test remove PACKAGE_NAME
```

Also run the repository's existing typecheck/build command when source compilation changed. If `dsh`, Node, or pnpm is unavailable, validate the files statically and report exactly which runtime proof was not run.

### 6. Publish only when requested

Before a commit or push, inspect `git status`, `git diff`, and recent log; stage only intended files. Follow `references/github-publishing.md` for repository creation, topics, release notes, and marketplace fields.

For a DSH plugin repository, always leave a copy-ready install command in the README:

```sh
dsh plugin --profile web add github:OWNER/REPO
```

Include a SHA-pinned form for reproducible or security-sensitive use.

## Hard Rules

- Official `deepseek-ai/deepseek-harness` docs and source outrank community templates.
- Do not invent `dsh.skills` or `dsh.mcpServers`; they are not current official package manifest fields.
- Do not describe profile patch `config` as a deep merge. A later patch replaces the row's complete `config`.
- Do not hide Git install-script risk: `prepare` runs package code outside agent tool approvals.
- Do not publish secrets, local paths, generated caches, or unrelated worktree changes.
- Do not claim GitHub Releases are required by DSH. They are a normal optional SemVer distribution channel.

## Completion Report

Return only the useful result:

- files created or changed;
- validation actually run and its result;
- local/GitHub install command;
- repository or release URL when published;
- one concrete blocker or version caveat, if any.
