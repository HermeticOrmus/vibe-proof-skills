<p align="center">
  <img src="https://ormus.solutions/mascot/pixellab_liquid_to_hylian_shield.gif" alt="Vibe Proof Skills" width="128" style="image-rendering: pixelated;" />
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
