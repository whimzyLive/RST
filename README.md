# RST — Rapid Stack

> High-level serverless constructs on AWS CDK. A community fork of [SST v2](https://v2.sst.dev).

[Contribute](CONTRIBUTING.md)

---

## What is RST?

**RST (Rapid Stack)** is a fork of [SST v2](https://github.com/sst/v2) — the open-source framework formerly known as **Serverless Stack**. RST continues the mission of providing **high-level, easy-to-use constructs** built on top of **AWS CDK** to help developers quickly spin up serverless architectures on AWS, while retaining the ability to drop down to raw CDK constructs for complex use cases.

## Motivation

### Why does this fork exist?

SST v2 was built on **AWS CDK** and offered a powerful, abstracted layer for deploying serverless applications. Developers loved it because:

- **Simplicity** — High-level constructs (`Api`, `Function`, `Bucket`, `Cron`, etc.) made common patterns trivial to set up.
- **Escape hatch** — For complex architectures, you could always fall back to the full power of AWS CDK.
- **Predictability** — Deployments went through **AWS CloudFormation**, giving teams the lifecycle rules, rollback behavior, drift detection, and stack management they relied on in production.

With **SST v3**, the team behind SST made the decision to **replace the underlying engine entirely** — moving from AWS CDK to a **Pulumi-based architecture**. While this enables multi-cloud support and aligns with Pulumi's broader provider ecosystem, it introduced several challenges for existing users:

1. **Loss of CloudFormation** — Teams that depend on CloudFormation's predictable deployment lifecycle, stack policies, and rollback guarantees lost those capabilities.
2. **No migration path** — Because v3 is a fundamentally different architecture and engine, there is no low-friction migration from v2 to v3. The SST team cannot provide one — the two versions are architecturally incompatible.
3. **Significant migration effort** — Applications and platforms built on SST v2 would require a full rewrite of their infrastructure code to adopt v3, which is a major undertaking for any team.
4. **AWS-first teams left behind** — Many companies and developers are committed to AWS and prefer the maturity, tooling, and operational model of CloudFormation-based deployments. SST v3's shift doesn't serve them.

### What RST aims to do

RST picks up where SST v2 left off. The goal is to:

- **Maintain and evolve** the AWS CDK-based construct library that SST v2 provided.
- **Continue offering high-level abstractions** that make serverless on AWS fast and approachable.
- **Preserve CloudFormation-based deployments** with all the lifecycle management, rollback behavior, and operational predictability that comes with it.
- **Serve the community** of developers and teams who built on SST v2 and need a supported path forward without a full rewrite.

### Why not just keep calling it SST?

RST is an independent project. We use a distinct name to respect the original SST project's licensing and branding, and to avoid any confusion with the official SST v3 and beyond. This is a community effort with its own identity and direction.

## Architecture

RST is a **TypeScript monorepo** built with:

| Technology | Role |
|---|---|
| **AWS CDK** | Infrastructure-as-code foundation |
| **AWS CloudFormation** | Deployment engine — predictable, rollback-capable |
| **TypeScript** | Primary language (strict ESM) |
| **Node.js 18** | Runtime target |
| **pnpm + Turborepo** | Monorepo management |
| **Vitest** | Testing framework |
| **esbuild** | Bundling |

### Project Layout

```
packages/sst/src/constructs/  — RST construct classes (public API)
packages/sst/src/node/        — Lambda runtime modules
packages/sst/src/cli/         — CLI commands
packages/sst/src/context/     — AsyncLocalStorage context system
packages/sst/support/         — esbuild-bundled support files
examples/                     — Example projects and starter templates
www/                          — Documentation site
```

## Getting Started

For documentation on available constructs and usage, see the [SST v2 docs](https://v2.sst.dev) (still applicable to RST).

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for details on how to get started.

## License

[MIT](LICENSE)
