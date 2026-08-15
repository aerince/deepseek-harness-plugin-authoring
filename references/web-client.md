# Web Client Plugins

Use this route only when the user explicitly needs browser UI. It is more version-sensitive than a host-only plugin.

## Current Package Contract

A browser package must declare both:

1. `dsh.client` with `platform: "web"`;
2. `exports["./client"]` pointing to the built browser bundle.

A combined installable bundle can use:

```json
{
  "type": "module",
  "main": "lib/index.js",
  "exports": {
    ".": {
      "types": "./lib/types/index.d.ts",
      "default": "./lib/index.js"
    },
    "./client": {
      "types": "./lib/types/client/index.d.ts",
      "default": "./lib/client.js"
    },
    "./package.json": "./package.json"
  },
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    },
    "client": {
      "inject": [
        "@deepseek-ai/dsh-client-runtime"
      ],
      "platform": "web"
    }
  }
}
```

List only real package dependencies needed for activation in `inject`. Add optional `immediately` only when the module must be prefetched at boot.

The official verifier treats these mismatches as errors:

- exporting `./client` without `dsh.client` means the browser half is never served;
- declaring `dsh.client` without exporting `./client` leaves no bundle to serve.

Metadata changes take effect after restart.

## Host Half

A UI-only package still needs a host entry so the Cordis Loader can activate the package:

```ts
export function apply(): void {
  // Browser behavior lives in ./client.
}
```

Its `cordis.patch.yml` inserts the package like any other bundle.

## Browser Half

The browser source normally exports its own `inject` and `apply(ctx)` for client-side Cordis services. Register UI through the current official client APIs, such as slot services, instead of writing directly into unrelated host DOM.

The built output is a CommonJS factory registered lazily with the runtime loader. The current official wrapper is equivalent to:

```js
window.__ModuleLoader__.load({
  id: "PACKAGE_NAME",
  factory: (require) => {
    var module = { exports: {} };
    var exports = module.exports;
    // Bundled CommonJS output.
    return module.exports;
  }
});
```

The exact wrapper/build preset is internal and may change. Before scaffolding it:

1. inspect the current official `packages/client/tsdown.client.ts`;
2. inspect one current official package with `dsh.client`;
3. inspect one current third-party plugin using the same DSH version;
4. reuse the target repository's bundler if possible.

Externalize React and DSH runtime packages expected from the host. Do not copy the official monorepo's `workspace:^` ranges or import its monorepo-local build preset.

## Dependency Boundaries

`dsh.client.inject` expresses browser activation edges by package name. Cross-plugin runtime value imports are brittle and can violate lazy module boundaries; collaborate through Cordis services. Type-only imports used for declaration merging are acceptable when the installed packages expose those types.

## Minimal Verification

Run the repository's build and then check:

- `lib/index.js` exists;
- `lib/client.js` exists;
- package exports resolve both `.` and `./client`;
- `dsh.client.platform` is `web`;
- every injected package is installed and composed;
- `dsh --profile <name> --dump-config` includes the host package;
- opening the Web UI produces no module-loader or browser-console errors.

If browser automation is available, exercise the actual UI once. Do not declare success from a build alone.
