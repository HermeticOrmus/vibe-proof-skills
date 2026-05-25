# Using this repo with Cursor

This project includes a **Cursor project rule** so the vibe-proof security checklist is available when you ask for a security review.

## In this repository

1. Open the folder in Cursor.
2. The rule [`.cursor/rules/vibe-proof.mdc`](.cursor/rules/vibe-proof.mdc) is committed with `alwaysApply: false`, so it applies on demand rather than on every turn. Reference it when you want a security review, or describe a security-review task and Cursor will pull it in.
3. In Cursor, confirm under **Settings, then Rules**, where `vibe-proof` should appear.

## Use the same checklist in another project

**Cursor (recommended)**: Copy `.cursor/rules/vibe-proof.mdc` into that project's `.cursor/rules/` directory (create the folders if needed). Keep `alwaysApply: false` so it stays a situational rule for security review, not a constant instruction.

**Other AI coding tools**: If a stack only supports a root instruction file, copy [`CLAUDE.md`](CLAUDE.md) into that project instead (or merge its contents into your existing instructions). Most modern AI coding tools (Claude Code, Continue, Cline, Windsurf, Aider) read a root-level instruction file.

## Optional: personal Agent Skills

If you want the same content as a reusable skill under `~/.cursor/skills`, use [`skills/vibe-proof/SKILL.md`](skills/vibe-proof/SKILL.md). Copy or symlink it into your personal skills directory.
