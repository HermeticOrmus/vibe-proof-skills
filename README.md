<p align="center">
  <img src="https://ormus.solutions/mascot/pixellab_liquid_to_philosopher_stone.gif" alt="Vibe Proof Skills" width="128" style="image-rendering: pixelated;" />
</p>

<h1 align="center">Vibe Proof Skills</h1>

<p align="center">
  <em>A Claude Code skill that hardens vibe-coded full-stack apps — parallel security audit across frontend, backend, and config, then fixes by severity.</em>
</p>

<p align="center">
  <a href="https://github.com/HermeticOrmus/vibe-proof-skills/stargazers"><img src="https://img.shields.io/github/stars/HermeticOrmus/vibe-proof-skills?style=flat-square&color=aa8142" alt="Stars" /></a>
  <a href="https://github.com/HermeticOrmus/vibe-proof-skills/blob/main/LICENSE"><img src="https://img.shields.io/github/license/HermeticOrmus/vibe-proof-skills?style=flat-square&color=aa8142" alt="License" /></a>
  <a href="https://github.com/HermeticOrmus/vibe-proof-skills/commits"><img src="https://img.shields.io/github/last-commit/HermeticOrmus/vibe-proof-skills?style=flat-square&color=aa8142" alt="Last Commit" /></a>
  <img src="https://img.shields.io/badge/Claude_Code-aa8142?style=flat-square&logo=anthropic&logoColor=white" alt="Claude Code" />
</p>

---

## The problem

A vibe-coded MVP ships fast and works on the demo. What it usually also ships: SQL injection through an unvalidated `ORDER BY`, a hardcoded backdoor password left in from "temporary" testing, API tokens passed in URL params, a `.env` file committed to git, and no security headers at all. These are not exotic bugs. They are the default state of an app that was built for the happy path and never security-reviewed.

This skill runs that review. It audits three layers in parallel, merges the findings into one prioritized list, and fixes them in severity order, building after each category so nothing regresses.

## The seven checks

| # | Check | Catches |
|---|-------|---------|
| 1 | Injection vectors | SQL injection, `eval`, unvalidated sort/filter columns, unbounded URL params |
| 2 | PII and secret exposure | Hardcoded passwords, secrets in URLs, `.env` in git, public env vars that should be private |
| 3 | Missing security headers | Absent HSTS, `nosniff`, `X-Frame-Options`, weak CSP |
| 4 | Error leakage | Stack traces in responses, `err.message` returned to clients, sensitive `console.log` |
| 5 | Input validation gaps | Unvalidated POST/PUT bodies, missing enum allowlists, extension-from-filename |
| 6 | Dead code and attack surface | Unused routes, GET-as-POST aliases, disabled-but-present features |
| 7 | Credential hygiene | Short session secrets, missing cookie flags, no rate limiting on sensitive endpoints |

The full checklist for each is in [`CLAUDE.md`](CLAUDE.md). Concrete before/after fixes are in [`EXAMPLES.md`](EXAMPLES.md).

## The process

1. **Parallel audit, read-only.** Three scans run at once: a frontend-security pass, a backend/API pass, and a config/credential pass. Each reports findings as `file:line`, severity, and a fix suggestion.
2. **Synthesize and prioritize.** Overlapping findings are deduplicated and sorted into one list by severity: CRITICAL, HIGH, MEDIUM, LOW.
3. **Fix by severity.** Work top-down. After each category, run `npm run build` (or the project equivalent) and verify no regressions.
4. **Remediate credentials.** If `.env` files were tracked, remove them from git and rotate every exposed credential.
5. **Verify post-deploy.** A passing build does not prove external services are reachable. Test each one with a minimal query after deploy.

Severity order:

| Priority | Category |
|----------|----------|
| CRITICAL | Backdoor passwords, injection, credential leaks, secrets in URLs |
| HIGH | PII exposure, missing validation, error leakage, missing HSTS, GET-as-POST |
| MEDIUM | Missing rate limits, enum validation, dead code, CSP tightening |
| LOW | Unused packages, `console.log`, config optimization |

## Install

### As a project CLAUDE.md

Drop [`CLAUDE.md`](CLAUDE.md) at the root of your repository. Claude Code picks it up automatically, then ask it to "security audit this project against CLAUDE.md, then fix by severity." Merge with existing project instructions if any.

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/HermeticOrmus/vibe-proof-skills/main/CLAUDE.md
```

### As a Claude Code skill

The same content is packaged as a skill under [`skills/vibe-proof/`](skills/vibe-proof/) for `~/.claude/skills/`. See the `SKILL.md` inside for installation.

### In Cursor

See [`CURSOR.md`](CURSOR.md) for the Cursor-rule equivalent at [`.cursor/rules/vibe-proof.mdc`](.cursor/rules/vibe-proof.mdc).

### In other AI coding tools

If your tool reads a single instruction file at the project root, copy `CLAUDE.md` to whatever name your tool expects (`AGENTS.md`, `INSTRUCTIONS.md`, etc.).

## See also

- [`mars-skills`](https://github.com/HermeticOrmus/mars-skills): the broader production-readiness companion, auditing the hidden sins that separate "works on my machine" from a system safe to run in production.
- [`vibe-engineer-skills`](https://github.com/HermeticOrmus/vibe-engineer-skills): the discipline of directing AI codegen well, hypothesis before help, scoped prompts, validate before accepting.

## Contributing

PRs welcome, especially for additional worked examples in [`EXAMPLES.md`](EXAMPLES.md), new fix patterns for other stacks, and adaptations of `CURSOR.md` for other AI coding tools (Windsurf, Cline, Aider, Continue, etc.).

## License

MIT. Use it, fork it, merge it into your own CLAUDE.md.

---

## Part of the Libre Open-Source Stack for Claude Code

This repository is part of a growing family of open-source toolkits for Claude Code.

### Libre suite — comprehensive plugin bundles

- [LibreUIUX-Claude-Code](https://github.com/HermeticOrmus/LibreUIUX-Claude-Code) — UI/UX development (152 agents, 70 plugins, 76 commands, 74 skills)
- [LibreArch-Claude-Code](https://github.com/HermeticOrmus/LibreArch-Claude-Code) — Software architecture and system design
- [LibreCopy-Claude-Code](https://github.com/HermeticOrmus/LibreCopy-Claude-Code) — Technical writing and documentation engineering
- [LibreDevOps-Claude-Code](https://github.com/HermeticOrmus/LibreDevOps-Claude-Code) — DevOps engineering and infrastructure automation
- [LibreEmbed-Claude-Code](https://github.com/HermeticOrmus/LibreEmbed-Claude-Code) — Embedded systems, firmware, and IoT development
- [LibreFinTech-Claude-Code](https://github.com/HermeticOrmus/LibreFinTech-Claude-Code) — Financial technology development
- [LibreGEO-Claude-Code](https://github.com/HermeticOrmus/LibreGEO-Claude-Code) — AI-search optimization (ChatGPT, Perplexity, Gemini, Google AI Overviews)
- [LibreGameDev-Claude-Code](https://github.com/HermeticOrmus/LibreGameDev-Claude-Code) — Game development across Godot, Unity, Unreal
- [LibreMLOps-Claude-Code](https://github.com/HermeticOrmus/LibreMLOps-Claude-Code) — ML engineering and AI operations
- [LibreMobileDev-Claude-Code](https://github.com/HermeticOrmus/LibreMobileDev-Claude-Code) — Mobile app development (Flutter, React Native, native iOS, native Android)
- [LibreSecOps-Claude-Code](https://github.com/HermeticOrmus/LibreSecOps-Claude-Code) — Security operations
- [LibreSessionFlow-Claude-Code](https://github.com/HermeticOrmus/LibreSessionFlow-Claude-Code) — Session lifecycle: handoff, pickup, absorb, explore, close

### Skills mini-repos — single CLAUDE.md drop-ins

- [vibe-engineer-skills](https://github.com/HermeticOrmus/vibe-engineer-skills) — Direct AI codegen well: hypothesis before help, scoped prompts, validate before accepting
- [markdown-discipline-skills](https://github.com/HermeticOrmus/markdown-discipline-skills) — Strip AI-slop from markdown (no em dashes, no marketing fluff)
- [shell-safety-skills](https://github.com/HermeticOrmus/shell-safety-skills) — `set -euo pipefail` discipline plus 15 failure-mode examples
- [commit-standard-skills](https://github.com/HermeticOrmus/commit-standard-skills) — Ormus Commit Standard v1.0 plus commit-msg hook and commitlint
- [unwoke-skills](https://github.com/HermeticOrmus/unwoke-skills) — Strip AI theater (ten sins to eliminate, symmetric engagement)
- [python-conventions-skills](https://github.com/HermeticOrmus/python-conventions-skills) — Modern Python 3.11+ (types, pathlib, async, ruff, mypy, uv)
- [typescript-conventions-skills](https://github.com/HermeticOrmus/typescript-conventions-skills) — TypeScript strict mode, discriminated unions, Result types
- [hermetic-laws-skills](https://github.com/HermeticOrmus/hermetic-laws-skills) — Seven Hermetic Principles applied to engineering
- [riper-workflow-skills](https://github.com/HermeticOrmus/riper-workflow-skills) — Research / Innovate / Plan / Execute / Review systematic dev
- [six-day-cycle-skills](https://github.com/HermeticOrmus/six-day-cycle-skills) — Sustainable shipping cadence with mandatory rest
- [token-optimization-skills](https://github.com/HermeticOrmus/token-optimization-skills) — Claude Code token and context optimization
- [osint-skills](https://github.com/HermeticOrmus/osint-skills) — OSINT research methodology (multi-wave investigative spiral)
- [calcinate-skills](https://github.com/HermeticOrmus/calcinate-skills) — Stage 1 of the Magnum Opus (burn project bloat)
- [claude-md-overhaul-skills](https://github.com/HermeticOrmus/claude-md-overhaul-skills) — Audit CLAUDE.md and MEMORY.md against caps
- [session-handoff-skills](https://github.com/HermeticOrmus/session-handoff-skills) — Session handoff and pickup discipline
- [naming-skills](https://github.com/HermeticOrmus/naming-skills) — Product naming methodology (mine the brand's vocabulary)
- [magnum-opus-skills](https://github.com/HermeticOrmus/magnum-opus-skills) — Seven-stage alchemy applied to project transformation
- [mem-search-skills](https://github.com/HermeticOrmus/mem-search-skills) — Search claude-mem cross-session memory: search, filter, fetch
- [hypothesis-debugging-skills](https://github.com/HermeticOrmus/hypothesis-debugging-skills) — Hypothesis-driven debugging: reproduce, isolate, test, fix
- [tdd-skills](https://github.com/HermeticOrmus/tdd-skills) — Test-driven development (Red-Green-Refactor) for JS/TS and Python
- [mars-skills](https://github.com/HermeticOrmus/mars-skills) — Production-readiness audit: the five mortal sins of vibe-coded MVPs
- [git-workflow-skills](https://github.com/HermeticOrmus/git-workflow-skills) — Clean git workflow: branch, atomic commits, reviewable PRs
- [code-review-skills](https://github.com/HermeticOrmus/code-review-skills) — Domain-aware code review: classify the code, then focus
- [code-comprehension-skills](https://github.com/HermeticOrmus/code-comprehension-skills) — Understand an unfamiliar codebase fast
- [dx-audit-skills](https://github.com/HermeticOrmus/dx-audit-skills) — Audit developer experience: docs, onboarding, tooling friction
- [setup-env-skills](https://github.com/HermeticOrmus/setup-env-skills) — Set up a project's development environment
- [automate-skills](https://github.com/HermeticOrmus/automate-skills) — Turn repetitive tasks into reliable automation scripts
- [quick-fix-skills](https://github.com/HermeticOrmus/quick-fix-skills) — Fast troubleshooting for common issues
- [prime-context-skills](https://github.com/HermeticOrmus/prime-context-skills) — Prime project context at the start of a session
- [auto-docs-skills](https://github.com/HermeticOrmus/auto-docs-skills) — Generate and maintain project documentation
- [learning-skills](https://github.com/HermeticOrmus/learning-skills) — Learn any technology: roadmaps, explanations, practice, cheatsheets, comparisons
- [linux-sysadmin-skills](https://github.com/HermeticOrmus/linux-sysadmin-skills) — Linux system administration: security, performance, diagnostics, monitoring, maintenance

### Template source

- [andrej-karpathy-skills](https://github.com/HermeticOrmus/andrej-karpathy-skills) — the canonical single-file CLAUDE.md pattern (fork of jiayuan_jy's original)

Star the family, not just one — that's how the suite stays coherent.
