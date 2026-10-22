# Official DeepSeek Harness Contract

## Freshness Baseline

Verified on 2026-10-22 against:

- Repository: <https://github.com/deepseek-ai/deepseek-harness>
- Commit: `687f282e8625c6b8d11ff58211a90524be2e4317`
- Release tag: `dsh-v0.2.1-rc.2` (published 2026-10-17)
- Source version: `0.2.1-rc.2`
- Status: latest published prerelease at verification time; developer preview with expected breaking changes

Check the current head before authoring:

```sh
gh api repos/deepseek-ai/deepseek-harness/commits/master --jq .sha
```

If it differs from the baseline, re-read only the official pages relevant to the task:

- First plugin: <https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/index.zh.md>
- Tools: <https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/tool.zh.md>
- Config: <https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/config.zh.md>
- Packaging: <https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/basic/publish.zh.md>
- Services: <https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/framework/service.zh.md>
- Events: <https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/develop/framework/events.zh.md>
- CLI installer: <https://github.com/deepseek-ai/deepseek-harness/blob/master/apps/cli/src/plugin.ts>
- Browser modules: <https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/subsystems/client-modules.md>

When offline, use this baseline and say that a current-head check was not possible.

## Host Plugin

A plugin is a TypeScript module whose exported `apply` receives a Cordis context:

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'my-plugin'

export function apply(ctx: Context) {
  // Register capabilities through ctx.
}
```

Supported shapes are named exports, a default object with `name`/`inject`/`apply`, and a class extending `Service`. Prefer named exports. Use a service class only to provide a service to other plugins.

Required dependencies go in `export const inject = ['tools']`. Cordis waits until they are ready. Optional dependencies are queried with `ctx.get('service-name')`.

Registrations made through `ctx` are removed when the plugin unloads. Manually owned resources use:

```ts
ctx.effect(() => {
  const resource = acquire()
  return () => resource.close()
})
```

## Local Source Development

An overlay patch is a YAML array:

```yaml
- insert:
    - id: hello
      name: '/absolute/path/to/plugin.ts'
```

The source tutorial requires an absolute path because the patch does not change the Loader's profile-based module resolution.

```sh
pnpm dsh web --patch ./scratch-plugin/cordis.yml
```

The official source checkout currently requires Node.js `^22.19.0 || >=24.0.0` and pins `pnpm@11.7.0`. A third-party package should verify its own minimums against its selected APIs rather than blindly copying every monorepo constraint.

## Tools

Tool plugins import `defineTool` from `@deepseek-ai/dsh-tools`, inject `tools`, and register through `ctx.tools.register(...)`.

- `parameters` infers and validates execution arguments.
- `execute` returns a canonical value matching `output.schema`.
- `output.render` maps that value to model-facing content.

Use current official examples for nested schemas, background work, policy hooks, Code Mode, or UI cards.

## Configuration

Export a TypeScript `interface Config` and a same-named Schemastery value:

```ts
import Schema from '@deepseek-ai/schemastery'

export interface Config {
  timeoutMs: number
}

export const Config: Schema<Config> = Schema.object({
  timeoutMs: Schema.number().default(30000),
})
```

Do not export a plain object as `Config`; it does not implement the required Standard Schema interface. Put defaults and constraints in the schema. Config changes unload and reload the plugin.

## Services and Events

A provider extends `Service` and calls `super(ctx, 'serviceName')`. Add its type to `Context` through TypeScript declaration merging. A required consumer lists the service in `inject`; it disposes if the provider disappears and reloads when the provider returns.

Cordis event modes are:

- `ctx.emit`: synchronous broadcast;
- `ctx.bail`: first meaningful result wins;
- `await ctx.serial`: ordered async dispatch with short circuit;
- `await ctx.waterfall`: wrapping pipeline; listeners must call `next()` unless intentionally intercepting.

Type custom events by declaration-merging `Events`. Persisted session records such as `tool/result` are not automatically same-named Cordis events; observe `session/event` and inspect `event.type` when needed.

## Installable Bundle

The package manifest must include:

```json
{
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  }
}
```

The patch is a YAML array, for example:

```yaml
- insert:
    - id: hello
      name: dsh-hello-plugin
```

Without `dsh.bundle`, `dsh plugin add` installs a dependency but activates no profile layer.

Declare a tested compatibility range with `engines.dsh`. DSH v0.2.1-rc.2 checks it during installation and profile startup; when it is absent, DSH falls back to the package's `@deepseek-ai/dsh-*` peer dependency ranges. Prefer `engines.dsh` when compatibility is defined by host APIs, and include only versions verified by the plugin.

The CLI entrypoint is a pnpm forwarding interface:

```sh
dsh plugin --profile demo add ./plugin
dsh plugin --profile demo add github:OWNER/REPO
dsh plugin --profile demo remove PACKAGE_NAME
```

Verify composition before starting:

```sh
dsh --profile demo --dump-config
dsh --profile demo
```

Layer order is profile bundles, profile `cordis.patch.yml`, `$DSH_HOME/cordis.patch.yml`, then argv `--patch` files in order. Later layers win. A patch replaces a target row's entire `config`, so restate every key that must survive.

## Git Source and Build Scripts

Git installation fetches source. An ordinary `build` script is not enough. A TypeScript package must either:

1. include built output in the installed source/archive; or
2. provide a self-contained `prepare` script.

pnpm 10+ blocks dependency build scripts until the installer authorizes the exact package under the profile's `pnpm-workspace.yaml` `allowBuilds`. Install scripts run with the user's process permissions and outside agent tool approvals. Recommend source review and a pinned commit:

```sh
dsh plugin --profile web add github:OWNER/REPO#COMMIT_SHA
```

When importing a DSH package whose instance must be shared with the host (for example `@deepseek-ai/dsh-tools` or `@deepseek-ai/schemastery`), declare it in both `peerDependencies` and `devDependencies` using published semver ranges. DSH v0.2.1-rc.2 resolves declared peers against the running host; keep independent third-party dependencies in `dependencies`.

Publishing prebuilt output to npm or installing a packed tarball avoids Git-time compilation.

## Removed or Unverified Formats

Do not use:

- the removed `.dsh-plugin` manifest;
- the removed repository-plugin wrapper/cache path;
- community-only `dsh.skills` metadata;
- community-only `dsh.mcpServers` metadata.

Current official package metadata verified for this Skill is `dsh.bundle` and, for browser modules, `dsh.client`.
