# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**solscan-skills** hosts [Agent Skills](https://agentskills.io/) for Solscan tooling. Each skill lives in `skills/<name>/` with a `SKILL.md` entry point and optional `references/` files.

- `skills/solscan-cli/` — teaches agents to drive the [`solscan` CLI](https://github.com/solscanofficial/solscan-cli) (Solscan Pro API v2.0): command shape, pagination/time-filter conventions, and an action index pointing into per-resource reference files (`references/<resource>.md`, mirroring `src/commands/*.js` in the CLI repo) plus response-shape docs (`references/responses/*.md`).

## Keeping Skills in Sync

The CLI source of truth is the separate `solscan-cli` repository. When a command is added, renamed, or its flags change there, update the matching `skills/solscan-cli/references/<resource>.md`, the action index in `SKILL.md`, and any affected `references/responses/*.md` so the skill doesn't drift from actual CLI behavior.

## Conventions

- Links between skill files are relative (e.g. `../SKILL.md`); links into CLI source use absolute GitHub URLs (`https://github.com/solscanofficial/solscan-cli/blob/main/...`).
- No build step or tests — the repo is Markdown only.
