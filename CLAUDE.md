# SST v2 — Agent Instructions

## Stack

TypeScript 5.2.2 | Node 18 | AWS CDK 2.201.0 | AWS SDK v3 3.699.0 | Vitest | pnpm + Turbo monorepo

## Critical Rules

### ESM (most common agent mistake)
- **Always** use `.js` extensions in imports: `import { Foo } from "./Foo.js"`
- `"type": "module"` throughout — no `require()` in source
- `export *` for public APIs, `export * as X` for config modules, `export type` for type-only

### TypeScript
- Target `esnext`, module `nodenext`, moduleResolution `nodenext`
- Mark internal APIs `/** @internal */`
- Use custom errors from `src/error.ts`, not raw `Error`
- Async/await only — no raw Promise chains

### CDK Constructs
- Extend CDK base classes: `constructor(scope: Construct, id: string, props?: Props)`
- Implement `SSTConstruct`: `getConstructMetadata()` + `getBindings()`
- Functional stacks: `(ctx: StackContext) => T` — use `use(OtherStack)` for dependencies (order matters)
- `setDefaultFunctionProps()` must be called before adding functions

### Runtime
- Proxies: `export const Api = /* @__PURE__ */ createProxy<ApiResources>("Api")` — `@__PURE__` required
- Env var schema: `SST_<ConstructName>_<constructId>_<propName>`
- Handlers: use `ApiHandler`, `SQSHandler` etc. — wraps `AsyncLocalStorage` context
- Never store request state in module-level variables

### Testing
- **Vitest** (not Jest): `import { test, expect, vi } from "vitest"`
- Tests validate CloudFormation output, not runtime: `hasResource(stack, type, props)`
- Each test: `const stack = new Stack(await createApp(), "stack")` — isolated per test
- Import helpers from `"./helper.js"`: `createApp`, `hasResource`, `countResources`, `ANY`, `ABSENT`, `objectLike`
- `countResources()` auto-adds +1 for MetadataUploader — expect this offset
- Test files: `[ConstructName].test.ts` matching source PascalCase

### Naming
- **PascalCase**: construct files (`Api.ts`, `Function.ts`)
- **kebab-case**: utility dirs (`event-bus/`, `websocket-api/`)
- **lowercase**: CLI files (`sst.ts`, `program.ts`)
- No default exports for constructs

### Project Layout
```
packages/sst/src/constructs/  — SST construct classes (public API)
packages/sst/src/node/        — Lambda runtime modules (kebab-case dirs)
packages/sst/src/cli/         — CLI commands
packages/sst/src/context/     — AsyncLocalStorage context system
packages/sst/support/         — esbuild-bundled support files
www/                           — Docusaurus docs site
```

### Build & Workflow
- `pnpm install` only — never npm
- Build: `node build.mjs && tsc` (esbuild first, then tsc to `dist/`)
- Publish from `dist/` directory
- Changesets for versioning: `pnpm version`, `pnpm release`
- Prettier + Husky pre-commit enforces formatting

### Don't Touch
- `seed.yml` — CI Node version pin
- AWS SDK version alignment (all clients at 3.699.0)
- CDK alpha must match CDK core version
- esbuild `createRequire` banner shim

Full context: `_bmad-output/project-context.md`
