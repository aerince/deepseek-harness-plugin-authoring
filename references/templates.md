# Minimal Plugin Templates

## Zero-Build GitHub Bundle

Use this default when the request does not require TypeScript types or a browser bundle.

```text
dsh-example-plugin/
├── package.json
├── index.js
├── cordis.patch.yml
├── README.md
└── LICENSE
```

`package.json`:

```json
{
  "name": "dsh-example-plugin",
  "version": "0.1.0",
  "description": "A minimal DeepSeek Harness plugin.",
  "type": "module",
  "main": "index.js",
  "exports": {
    ".": "./index.js",
    "./package.json": "./package.json"
  },
  "files": [
    "index.js",
    "cordis.patch.yml",
    "README.md",
    "LICENSE"
  ],
  "engines": {
    "node": "^22.19.0 || >=24.0.0"
  },
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/OWNER/REPO.git"
  },
  "keywords": [
    "deepseek-harness",
    "dsh",
    "dsh-plugin",
    "cordis"
  ],
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  }
}
```

`index.js`:

```js
export const name = 'example-plugin'

export function apply() {
  console.log('[example-plugin] loaded')
}
```

`cordis.patch.yml`:

```yaml
- insert:
    - id: example-plugin
      name: dsh-example-plugin
```

Keep the package name, patch `name`, repository URL, and README install command consistent.

## Local TypeScript Overlay

Use this while developing inside a DSH source checkout:

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'example-plugin'

export function apply(ctx: Context) {
  ctx.effect(() => {
    const timer = setInterval(() => console.log('[example-plugin] heartbeat'), 5000)
    return () => clearInterval(timer)
  })
}
```

```yaml
- insert:
    - id: example-plugin
      name: '/absolute/path/to/src/index.ts'
```

Do not add a standalone TypeScript build until distribution needs it.

## Tool Plugin

Use inside a DSH-aware TypeScript package after resolving current published dependency versions. Never copy monorepo-only `workspace:^` ranges into a third-party repository.

```ts
import type { Context } from '@deepseek-ai/cordis'
import { defineTool } from '@deepseek-ai/dsh-tools'

export const name = 'greet-tool'
export const inject = ['tools']

export function apply(ctx: Context) {
  ctx.tools.register(defineTool({
    name: 'greet',
    description: 'Greet someone by name.',
    parameters: {
      name: { type: 'string', required: true, description: 'The name to greet' },
    },
    output: {
      schema: { type: 'string' },
      render: (_args, value) => [{ type: 'text', text: value }],
    },
    async execute(args) {
      return `Hello, ${args.name}!`
    },
  }))
}
```

## Configurable Plugin

```ts
import type { Context } from '@deepseek-ai/cordis'
import Schema from '@deepseek-ai/schemastery'

export const name = 'configurable-plugin'

export interface Config {
  greeting: string
  timeoutMs: number
}

export const Config: Schema<Config> = Schema.object({
  greeting: Schema.string().default('Hello'),
  timeoutMs: Schema.number().default(30000),
})

export function apply(_ctx: Context, config: Config) {
  console.log(config.greeting, config.timeoutMs)
}
```

Patch config:

```yaml
- insert:
    - id: configurable-plugin
      name: dsh-configurable-plugin
      config:
        greeting: 'Hi'
        timeoutMs: 10000
```

If a later overlay changes this row's config, it must restate both keys because row config is replaced, not deep-merged.

## Service Provider

```ts
import { Service, type Context } from '@deepseek-ai/cordis'

declare module '@deepseek-ai/cordis' {
  interface Context {
    metrics: MetricsService
  }
}

export default class MetricsService extends Service {
  constructor(ctx: Context) {
    super(ctx, 'metrics')
  }

  record(event: string, value: number) {
    console.log(event, value)
  }
}
```

Required consumer:

```ts
export const inject = ['metrics']

export function apply(ctx: Context) {
  ctx.metrics.record('plugin_loaded', 1)
}
```

## Typed Event

```ts
import type { Context } from '@deepseek-ai/cordis'
import '@deepseek-ai/cordis'

declare module '@deepseek-ai/cordis' {
  interface Events {
    'example/ready': (payload: { id: string }) => void
  }
}

export function apply(ctx: Context) {
  ctx.on('example/ready', ({ id }) => console.log(`${id} is ready`))
  ctx.emit('example/ready', { id: 'worker-1' })
}
```

## TypeScript Distribution Choice

Use one route, not both:

- Prebuilt route: set `main` to `lib/index.js`, include `lib` in `files`, and ensure the Git commit/tarball contains it.
- Git-build route: add a self-contained `prepare` that builds `lib`; document the installer's required `allowBuilds` entry.

Do not invent a build command. Reuse the repository's installed bundler or compiler. Check registry versions before adding DSH dependencies; official monorepo manifests use workspace ranges that are invalid in an external package.

## README Install Block

````md
## Install

```sh
dsh plugin --profile web add github:OWNER/REPO
```

Pinned install:

```sh
dsh plugin --profile web add github:OWNER/REPO#COMMIT_SHA
```

Verify:

```sh
dsh --profile web --dump-config
```
````
