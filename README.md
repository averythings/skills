# VoltAgent Agent Skills

Official agent skills for coding agents working with the VoltAgent framework.

## Install

If your agent supports add-skill:

```bash
npx skills add VoltAgent/skills
```

## Skill List

- create-voltagent: Project setup guide with CLI and manual steps.
- voltagent-best-practices: Architecture and usage patterns for agents, workflows, memory, and servers.
- voltagent-core-reference: Reference for the VoltAgent class options and lifecycle methods.
- voltagent-docs-bundle: Lookup embedded docs from @voltagent/core/docs for version-matched documentation.
- indie-launch-checklist: 30-minute pre-ship technical readiness checklist for solo builders.
- llm-cost-guardrail: Runtime cost caps, per-user budgets, model-tier routing, and kill switch for LLM endpoints.

See [`GAP_ANALYSIS.md`](./GAP_ANALYSIS.md) for the reasoning behind the indie-focused skills.

## Manual Install

```bash
git clone https://github.com/VoltAgent/skills.git
```

Then configure your coding agent to load skills from the cloned directory.

## Links

- https://voltagent.dev/docs
- https://github.com/voltagent/voltagent
- https://agentskills.io

## License

MIT - see LICENSE.
