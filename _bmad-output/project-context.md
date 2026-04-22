---
project_name: 'SST v2 (Rapid Stack)'
user_name: 'Rushi Patel'
date: '2026-04-22'
sections_completed: ['technology_stack', 'language_rules', 'framework_rules', 'testing_rules', 'quality_rules', 'workflow_rules', 'anti_patterns']
status: 'complete'
rule_count: 67
optimized_for_llm: true
---

# Project Context for AI Agents

_This file contains critical rules and patterns that AI agents must follow when implementing code in this project. Focus on unobvious details that agents might otherwise miss._

---

## Technology Stack & Versions

- **TypeScript** 5.2.2 — Primary language, strict ESM (`"type": "module"`)
- **Node.js** 18 — Runtime target (tsconfig extends `@tsconfig/node18`)
- **AWS CDK** 2.201.0 — Infrastructure-as-code base for all constructs
- **AWS SDK v3** 3.699.0 — All AWS service clients (CloudFormation, S3, Lambda, SSM, STS, etc.)
- **esbuild** — Bundler for support runtime files (ESM output with `.mjs`)
- **Vitest** ^0.33.0 — Test runner (30s default timeout)
- **Turbo** 1.8.3 — Monorepo build orchestration
- **pnpm** — Package manager with workspace protocol
- **Prettier** ^2.8.4 — Code formatting (with Husky pre-commit hooks)
- **Changesets** ^2.26.0 — Version management and changelogs
- **Docusaurus** v2 — Documentation site at `www/` (deployed to v2.sst.dev)

### Version Constraints

- AWS SDK clients must stay aligned at the same minor version (currently 3.699.0)
- CDK alpha packages (`@aws-cdk/aws-lambda-python-alpha`) must match CDK core version
- `seed.yml` pins Node.js 14.17.0 for CI compilation — agents should not change this

## Critical Implementation Rules

### Language-Specific Rules

#### ESM Module System (Critical)
- ALL imports must use `.js` file extensions — even when importing `.ts` files (e.g., `import { Foo } from "./Foo.js"`)
- The project uses `"type": "module"` — no `require()` calls in source code
- For bundled support files that need `require`, esbuild injects a `createRequire` shim in the banner
- Use `export *` from index files to re-export public APIs
- Use `export * as Namespace` for config-like modules (e.g., `export * as Config from "./Config.js"`)

#### TypeScript Configuration
- Target: `esnext`, Module: `nodenext`, Module Resolution: `nodenext`
- `declaration: true` — all packages emit `.d.ts` type declarations
- `jsx: "react"` is enabled (for construct JSX usage)
- `allowSyntheticDefaultImports: true` — use default imports for CJS interop

#### Type Patterns
- Export types and interfaces alongside classes for public API surfaces
- Use `type` keyword for type-only exports: `export type { SSTConfig } from "./project.js"`
- Use union types for flexible props: `FunctionProps | ((stack: Stack) => FunctionProps)`
- Mark internal APIs with `/** @internal */` JSDoc — these should not be used by consumers

#### Error Handling
- Project defines custom error types in `src/error.ts` — use these instead of raw `Error`
- Async/await is the standard pattern — no raw Promise chains

### Framework-Specific Rules (SST + AWS CDK)

#### CDK Construct Pattern
- All SST constructs extend CDK base classes (e.g., `Api extends HttpApi`, `Function extends CDKFunction`)
- Constructor signature is always: `constructor(scope: Construct, id: string, props?: Props)`
- Props interfaces nest CDK types with SST-specific overrides on top
- Every SST construct must implement `SSTConstruct` interface: `getConstructMetadata()` and `getBindings()`

#### Functional Stack Pattern
- Stacks are defined as functions: `(ctx: StackContext) => T` where `StackContext = { app, stack }`
- Use `use(OtherStack)` for cross-stack dependency resolution — the referenced stack must be initialized first
- Stack IDs are auto-prefixed with app/stage via `app.logicalPrefixedName(id)`
- `setDefaultFunctionProps()` must be called before any functions are added to the stack

#### Runtime Proxy Pattern (Critical)
- Runtime modules use `createProxy<T>("ConstructName")` for environment variable resolution
- Always annotate proxies with `/* @__PURE__ */` for tree-shaking: `export const Api = /* @__PURE__ */ createProxy<ApiResources>("Api")`
- Environment variables follow the schema: `SST_<ConstructName>_<constructId>_<propName>`
- If a construct isn't bound to a function, the proxy throws — agents must set up bindings correctly

#### Handler & Context Pattern
- Lambda handlers use `Handler(type, callback)` wrapper for async-local context isolation
- Context uses `AsyncLocalStorage` — never store request state in module-level variables
- Use `RequestContext.with()` to set context, `RequestContext.use()` to read it
- Specific handler factories exist: `ApiHandler`, `SQSHandler`, etc. — use these instead of raw `Handler`
- `useEvent()` is a memoized accessor for the current Lambda event

#### Metadata System
- Every construct exposes `getConstructMetadata()` returning `{ type, data }` for SST Console reflection
- `isSSTConstruct()` type guard checks for metadata interface compliance

### Testing Rules

#### Framework & Runner
- **Vitest** (not Jest) — use `import { test, expect, vi } from "vitest"`
- Global timeout: 30 seconds (`vitest.config.ts`)
- Run tests: `vitest run` from `packages/sst/`
- Test directory: `packages/sst/test/`

#### Test Structure Pattern
- Each test creates a fresh isolated app: `const stack = new Stack(await createApp(), "stack")`
- `createApp()` provisions a unique temp directory per test — never share state between tests
- Tests validate **CloudFormation template output**, not runtime behavior
- Organize tests with comment section headers: `///////////////// Test Constructor /////////////////`

#### Custom Test Helpers (from `test/constructs/helper.ts`)
- `createApp(props?)` — Isolated app factory with test stage/region
- `hasResource(stack, type, props)` — Assert resource exists with partial props match
- `countResources(stack, type, count)` — Assert exact resource count (auto-adjusts +1 for internal MetadataUploader Lambda/IAM)
- `hasResourceTemplate(stack, type, props)` — Full resource template match
- `hasOutput(stack, logicalId, props)` / `hasNoOutput(stack, logicalId)` — CloudFormation output assertions
- `printResource(stack, type)` — Debug helper for inspecting generated resources

#### CDK Assertion Matchers
- `ANY` — Matches any value
- `ABSENT` — Asserts property does not exist
- `objectLike(props)` — Partial object matching
- `arrayWith(values)` — Array contains specific elements
- `stringLikeRegexp(pattern)` — Regex string matching
- Always import matchers from `"./helper.js"` (not directly from CDK)

#### Test Naming
- Test files: `[ConstructName].test.ts` (PascalCase matching source)
- Test descriptions: `"category: specific behavior"` (e.g., `"constructor: httpApi is undefined"`)
- Token values matched with regex: `expect(api.url).toMatch(/^\${Token\[TOKEN\.\d{4}\]}$/)`

### Code Quality & Style Rules

#### Formatting
- **Prettier** enforces formatting — run via `prettier --cache -w .` at root
- Husky pre-commit hook runs `lint-staged` which applies Prettier and ESLint fixes automatically
- Target files: `*.{js,ts,css,json,md}`

#### File & Folder Naming
- **PascalCase** for construct source files: `Api.ts`, `Function.ts`, `NextjsSite.ts`
- **kebab-case** for utility directories and non-construct files: `event-bus/`, `websocket-api/`, `static-file-list.ts`
- **lowercase** for CLI and internal tooling: `sst.ts`, `program.ts`, `colors.ts`
- Test files mirror source: `Api.test.ts` matches `Api.ts`

#### Code Organization
- `packages/sst/src/constructs/` — All SST construct classes (public API)
- `packages/sst/src/node/` — Runtime modules for Lambda (kebab-case dirs with `index.ts`)
- `packages/sst/src/cli/` — CLI commands and local dev tooling
- `packages/sst/src/context/` — AsyncLocalStorage-based context system
- `packages/sst/src/util/` — Internal utilities
- `packages/sst/support/` — Bundled support files (esbuild targets)

#### Export Conventions
- Index files use `export *` for public APIs — never use default exports for constructs
- Namespace exports for config modules: `export * as Config from "./Config.js"`
- Type-only exports use `export type` keyword explicitly
- Package `exports` field in `package.json` defines all public entry points — respect the export map

#### Documentation
- Public APIs use JSDoc comments
- Internal APIs marked with `/** @internal */` — do not expose these to consumers
- No excessive inline comments — code should be self-documenting

### Development Workflow Rules

#### Monorepo Structure
- **Turbo** orchestrates builds — `turbo run build` from root respects dependency graph
- Pipeline: `clean` → `cdk-version-check` → `build` (outputs to `dist/`)
- pnpm workspaces: `packages/*` and `www/`
- Never run `npm install` — use `pnpm install` exclusively

#### Build Pipeline
- `packages/sst`: `node build.mjs && tsc` — esbuild bundles support files first, then TypeScript compiles
- esbuild outputs ESM (`.mjs`) for Node.js runtime support files
- TypeScript emits declarations to `dist/` — this is the publishable directory
- `publishConfig.directory: "dist"` — npm publishes from `dist/`, not root

#### Git & Contributing Workflow
- Branch: `master` is the default branch
- Contribution flow: Issue → Spec (posted to issue) → PR → Core team review → Merge
- Specs must be written **before** implementation begins (see `CONTRIBUTING.md`)
- Specs should include: research, implementation steps, test cases, doc updates

#### Version Management
- **Changesets** manages versioning and changelogs
- Scripts: `pnpm version`, `pnpm release`, `pnpm release-snapshot`
- Each package has its own `CHANGELOG.md`

#### Local Development
- `tsc -w` for watch mode during development (`packages/sst`)
- Tests: `vitest run` from `packages/sst/`
- Formatting: `pnpm prettier` from root

### Critical Don't-Miss Rules

#### Anti-Patterns to Avoid
- **NEVER omit `.js` extensions** in imports — TypeScript compiles but runtime will fail with ESM resolution errors
- **NEVER use `require()`** in source code — this is a pure ESM project
- **NEVER use default exports** for constructs — always use named exports with `export *` re-exports
- **NEVER store request state in module-level variables** — use `AsyncLocalStorage` context pattern instead
- **NEVER create a construct without the `(scope, id, props)` signature** — CDK will fail to synthesize
- **NEVER modify `seed.yml`** — CI compilation pinning is intentional

#### Edge Cases Agents Must Handle
- `countResources()` helper auto-adds +1 for the internal MetadataUploader Lambda and IAM role — if your count is off by 1, this is why
- CDK Token values are not real strings — match them with regex patterns like `/^\${Token\[TOKEN\.\d{4}\]}$/`
- Stack dependency order matters: calling `use(StackA)` before `StackA` is initialized throws at synth time
- The `createProxy` pattern throws at runtime if bindings are missing — always verify `bind` configuration in constructs

#### Security Rules
- AWS credentials are resolved via `src/credentials.ts` — never hardcode credentials or account IDs
- IAM policies in tests use `lambdaDefaultPolicy` patterns — match these for consistency
- SDK client versions must stay aligned — mismatched versions can cause signature errors

#### Performance Gotchas
- `/* @__PURE__ */` annotations are required on proxy exports for tree-shaking — omitting them bloats bundles
- esbuild `banner` injection for `createRequire` is essential for CJS interop in ESM bundles — don't remove it
- Turbo caches build outputs in `dist/` — if builds seem stale, run `turbo run clean` first

---

## Usage Guidelines

**For AI Agents:**
- Read this file before implementing any code
- Follow ALL rules exactly as documented
- When in doubt, prefer the more restrictive option
- Update this file if new patterns emerge

**For Humans:**
- Keep this file lean and focused on agent needs
- Update when technology stack changes
- Review quarterly for outdated rules
- Remove rules that become obvious over time

Last Updated: 2026-04-22
