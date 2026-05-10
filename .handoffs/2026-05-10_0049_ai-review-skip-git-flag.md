# Session Handoff: ai-review v1.1.0 — Codex `--skip-git-repo-check` mandate
**Date**: 2026-05-10 00:49
**Branch**: main
**Working Directory**: /Users/michalblaha/Documents/Dev/Dev Projects/michalblaha-ai-plugins

## What Was Done

- Diagnosed startup error `SessionStart:startup hook error / Failed to run: ToolUseContext is required for prompt hooks. This is a bug.`
  - Root cause: `prompt`-type SessionStart hook in `wiki-tools` plugin (v1.3.0) hits a known Claude Code bug. Hook author documents it explicitly in `hooks/README.md`. Hot cache still loads via the parallel `command`-type hook, so impact is cosmetic. User chose "do nothing".
- Updated `plugins/ai-review/skills/double-cross-check/SKILL.md`:
  - Section 2: added two new paragraphs — (a) `--skip-git-repo-check` is mandatory on every `codex exec` call (rationale: skill never writes via Codex, only reads `--output-last-message`; CWD from Claude is non-deterministic), (b) fallback path = delegate to `codex:codex-rescue` on failure, then switch provider.
  - Section 6 (Codex / OpenAI): both example invocations updated to include `--skip-git-repo-check`. Added cross-ref note to section 2.
  - Section 7: updated `codex exec` calls in *Cross-reference strukturovaného datasetu* and *Bezpečnostní audit* examples.
  - Linter/user added `--ask-for-approval never` to section 6 examples (keep — intentional, complements the skip-git-repo-check change).
- Verified `--skip-git-repo-check` is the official Codex CLI flag (`codex exec --help`: *"Allow running Codex outside a Git repository"*). No alternative mechanism exists in Codex CLI for disabling that check.
- Bumped `ai-review` plugin version `1.0.0` → `1.1.0` in both manifests:
  - `plugins/ai-review/.claude-plugin/plugin.json`
  - `plugins/ai-review/.codex-plugin/plugin.json`
- Committed as `cabf4ab` on `main` and pushed to `origin/main`.
- After push, user ran `/plugin` (1 plugin bumped) and `/reload-plugins` (7 plugins, 10 skills, 8 agents, 8 hooks reloaded). New skill version is now active in this Claude Code session.

## Current State

Working tree clean. Branch `main` is in sync with `origin/main`. The `ai-review` plugin is at v1.1.0 locally and on GitHub. The `double-cross-check` skill now mandates `--skip-git-repo-check` on Codex calls and documents `codex:codex-rescue` as the failure fallback.

### Git State
- **Modified**: (none)
- **Untracked**: (none, aside from `.handoffs/` files which are tracked separately)
- **Recent commits**:
  - `cabf4ab` — ai-review v1.1.0: require --skip-git-repo-check on Codex calls
  - `ef6a8fc` — Fix Codex marketplace metadata
  - `296ea72` — Fix Codex marketplace: add 'name' field to each plugin entry
  - `0bcbc0b` — Add Codex CLI marketplace at .agents/plugins/marketplace.json
  - `090e01e` — Fix plugin.json: rely on auto-discovery instead of explicit paths

## What Remains

- [ ] (Optional) Apply the same `--skip-git-repo-check` + `codex:codex-rescue` fallback pattern to `plugins/ai-review/skills/second-opinion/SKILL.md` for consistency. Not done in this session — only `double-cross-check` was in scope.
- [ ] (Optional) Consider releasing a `wiki-tools` patch that drops the broken `prompt`-type SessionStart hook (or guards it behind a config flag) until the Claude Code `ToolUseContext` bug is fixed upstream. User opted to leave it alone for now.
- [ ] No CHANGELOG file exists for `ai-review`. If you adopt one later, backfill v1.1.0 entry: "Mandate `--skip-git-repo-check` on Codex calls; document `codex:codex-rescue` fallback."

## Key Decisions Made

- **Bump = minor (1.0.0 → 1.1.0)**, not patch. Rationale: not a bug fix, it's a behavioral mandate (every Codex call now requires a flag) and adds a fallback path. Not breaking — old call patterns still work, just with reduced robustness.
- **Don't fix wiki-tools SessionStart bug**: user explicitly said "nic nedělat". Hot cache still functions via the parallel command hook; the error is cosmetic.
- **`codex:codex-rescue` as fallback, not primary**: rescue subagent is a heavier orchestration path (invokes `codex-companion.mjs`). Use only after a direct `codex exec` fails. Single retry, then switch provider.
- **Marketplace files don't carry version**, so they were not touched. Only the two `plugin.json` manifests were bumped.

## Gotchas / Notes

- `wiki-tools` SessionStart `prompt`-type hook will keep emitting the red `ToolUseContext` error at every session start until either Claude Code or the plugin upstream is fixed. Treat as known noise.
- Codex docs note: when adding `--output-schema`, every nested object must declare `"additionalProperties": false` AND list every property in `"required"`. Otherwise Codex silently rejects the schema. This was already documented in section 6, just flagging.
- The `--ask-for-approval never` flag added by linter is intentional — pairs with `--skip-git-repo-check` to make calls fully non-interactive.
- User ran `/reload-plugins` after the push, so the new SKILL.md is live in this session — any future Codex call from `double-cross-check` *should* already use the flag without further intervention.

## Key Files

- `plugins/ai-review/skills/double-cross-check/SKILL.md` — the touched skill; sections 2, 6, 7 are where the mandate lives
- `plugins/ai-review/.claude-plugin/plugin.json` — v1.1.0
- `plugins/ai-review/.codex-plugin/plugin.json` — v1.1.0 (Codex marketplace manifest)
- `plugins/ai-review/skills/second-opinion/SKILL.md` — NOT touched this session; candidate for the same treatment
- `~/.claude/plugins/cache/openai-codex/codex/1.0.4/skills/codex-cli-runtime/SKILL.md` — internal contract for `codex:codex-rescue`; useful reference if revisiting fallback wiring
- `~/.claude/plugins/cache/michalblaha-ai-plugins/wiki-tools/1.3.0/hooks/hooks.json` — broken SessionStart prompt hook source (do not edit, gets overwritten on update)
