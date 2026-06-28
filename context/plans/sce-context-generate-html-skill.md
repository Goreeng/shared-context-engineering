# Plan: sce-context-generate-html-skill

## Change summary

Add a new generated SCE skill named `sce-context-generate-html` that helps an assistant produce human-readable HTML documentation from the current `context/` state. The skill must require a `sce-context-sync` pass before generation, read the refreshed project context, and write documentation under `context/html/` with CSS and correctly rendered diagrams to make the project easier for humans to understand.

The repository's generated config model makes `config/.opencode/**`, `config/automated/.opencode/**`, and `config/.claude/**` generated-owned outputs. The implementation should therefore update canonical Pkl sources and metadata first, then regenerate generated skill files rather than hand-editing generated outputs.

## Success criteria

- A new generated skill slug `sce-context-generate-html` exists in `config/.opencode/skills/sce-context-generate-html/SKILL.md`.
- The skill body instructs the agent to load and run `sce-context-sync` before building HTML docs.
- The skill body defines a deterministic output location under `context/html/`, with `context/html/index.html` as the default human-readable entrypoint.
- The skill body requires the generated HTML to include:
  - project overview content based on existing context files,
  - inline or local CSS for readability,
  - at least one diagram when helpful for project comprehension,
  - diagram rendering checks so diagrams are visibly rendered in a browser, not left as raw unsupported syntax.
- Canonical Pkl sources and per-target metadata are updated so generated-output parity remains deterministic.
- Generated outputs are refreshed from canonical sources.
- Context documentation reflects the new current-state skill/generation surface where relevant.
- Final validation runs `nix run .#pkl-check-generated` and `nix flake check`.

## Constraints and non-goals

- Do not hand-edit generated outputs as the source of truth; update canonical Pkl content and regenerate.
- Do not add a Rust CLI command for HTML generation in this plan; this is a skill/instruction addition only.
- Do not introduce a new npm/Bun/Rust dependency unless a later task explicitly justifies and scopes it.
- Keep the skill deterministic and reviewable; it should generate static files under `context/html/` and avoid opaque build steps.
- Do not persist narrative completed-work summaries in core context files; update context as current-state documentation only.
- Existing generated target behavior should remain unchanged except for adding the new skill and required metadata.

## Task stack

- [x] T01: `Run context freshness pass for generated-skill baseline` (status:done)
  - Task ID: T01
  - Goal: Run the existing `sce-context-sync` workflow before adding the new skill so planning/implementation starts from current context truth.
  - Boundaries (in/out of scope): In - load/follow `sce-context-sync`, inspect relevant generated-config and context files, repair focused stale context if found. Out - adding the new HTML skill or regenerating config outputs.
  - Done when: Relevant root/domain context files are verified against code truth, any required focused context repairs are committed to `context/`, and the implementer records whether the upcoming change is root-important or verify-only.
  - Verification notes (commands or checks): Follow the `sce-context-sync` checklist; verify `context/architecture.md`, `context/patterns.md`, `context/context-map.md`, and relevant generated-config context references align with current code/source files.
  - Completion evidence (2026-06-26): Ran `sce-context-sync` freshness review against generated-config context. Repaired focused stale context in `context/architecture.md` (manual + automated OpenCode + Claude target-tree wording; removed stale Claude TypeScript plugin-output claim), `context/sce/opencode-agent-trace-plugin-runtime.md` (aligned OpenCode event names with `message.updated` / `message.part.updated` code truth), `context/context-map.md` (updated OpenCode plugin runtime summary), and `context/overview.md` (active hook-intake wording). Targeted verification: `git diff --check -- context/overview.md context/architecture.md context/sce/opencode-agent-trace-plugin-runtime.md context/context-map.md context/plans/sce-context-generate-html-skill.md` passed; `test ! -d config/.claude/plugins && test ! -d .claude/plugins` passed; targeted `rg` stale-wording check over edited context files returned no matches; targeted generated-config search confirmed `config/pkl/generate.pkl` emits OpenCode plugin outputs only for `config/.opencode/plugins/sce-agent-trace.ts` and `config/automated/.opencode/plugins/sce-agent-trace.ts`; edited context files are under 250 lines. Attempted `nix run .#pkl-check-generated`, but this environment has no `nix` executable (`zsh:1: command not found: nix`), so full Nix parity/check validation remains for a Nix-equipped session.
  - Context-sync classification for upcoming change: root-important. Adding a generated SCE skill changes the cross-target generated skill surface and `context/html/` documentation convention, so later tasks should update durable root/domain context rather than treating the feature as verify-only.

- [x] T02: `Author canonical HTML-doc skill contract` (status:done)
  - Task ID: T02
  - Goal: Add the `sce-context-generate-html` skill body to the canonical Pkl skill source with instructions for synced-context HTML documentation generation.
  - Boundaries (in/out of scope): In - add one focused `UnitSpec` in the appropriate canonical shared-content module, covering trigger, required pre-sync, context inputs, `context/html/index.html` output, CSS/readability expectations, diagram expectations, and diagram-render verification. Out - renderer wiring, metadata tables, generated files, or context documentation updates.
  - Done when: The canonical skill body is present, names the skill `sce-context-generate-html`, requires loading/running `sce-context-sync` first, and defines static HTML output under `context/html/` with diagram rendering requirements.
  - Verification notes (commands or checks): Review the new Pkl text against existing skill style in `config/pkl/base/shared-content-code.pkl`; ensure the body is concise, deterministic, and does not require unavailable tools or new dependencies by default.
  - Completion evidence (2026-06-26): Added `sce-context-generate-html` as a focused `UnitSpec` in `config/pkl/base/shared-content-code.pkl`. The canonical body requires loading/running `sce-context-sync` before HTML generation, reads refreshed `context/` inputs, writes static docs under `context/html/` with `context/html/index.html` as the entrypoint, requires local/inline CSS for readability, and requires diagrams to render visibly in browser-compatible HTML instead of remaining raw unsupported syntax. Targeted verification: `rg -n "sce-context-generate-html|context/html/index.html|sce-context-sync|Diagram requirements" config/pkl/base/shared-content-code.pkl` found the expected slug/output/sync/diagram text; `pkl eval config/pkl/base/shared-content-code.pkl` passed with Pkl 0.31.1; `git diff --check -- config/pkl/base/shared-content-code.pkl` passed. Context-sync done gate verified root context files against this T02 slice and made no additional durable context edits because the skill is not yet wired into generated metadata/outputs; line counts for `context/overview.md`, `context/architecture.md`, `context/glossary.md`, `context/patterns.md`, and `context/context-map.md` remain below 250. `nix` remains unavailable in this environment, so Nix parity checks remain for later tasks with a Nix-equipped session.
  - Context-sync classification: root-important for the overall plan because a generated skill changes the cross-target generated skill surface and introduces the `context/html/` documentation convention. For this T02-only slice, durable user-facing context updates remain deferred to T05 because the skill is not wired into generated target metadata or outputs yet.

- [x] T03: `Wire skill into generated target metadata` (status:done)
  - Task ID: T03
  - Goal: Register the new skill slug in shared skill aggregation and all required target metadata tables so renderers can emit it deterministically.
  - Boundaries (in/out of scope): In - update `config/pkl/base/shared-content.pkl`, `config/pkl/base/shared-content-automated.pkl` if needed for automated OpenCode parity, and target metadata descriptions/compatibility coverage for OpenCode/automated/Claude according to the current renderer architecture. Out - changing renderer architecture or adding command frontmatter.
  - Done when: Every renderer that iterates shared skills has a description entry for `sce-context-generate-html`, metadata coverage remains complete, and the skill is included in the intended generated target trees.
  - Verification notes (commands or checks): Run targeted Pkl evaluation/metadata coverage checks if available, then include `nix run .#pkl-check-generated` in final validation.
  - Completion evidence (2026-06-26): Registered `sce-context-generate-html` in manual and automated shared skill aggregation (`config/pkl/base/shared-content.pkl`, `config/pkl/base/shared-content-automated.pkl`), added automated profile canonical skill body coverage in `config/pkl/base/shared-content-automated-code.pkl`, and added description metadata coverage in `config/pkl/renderers/common.pkl`, `opencode-metadata.pkl`, `opencode-automated-metadata.pkl`, and `claude-metadata.pkl`. Targeted verification: `pkl eval config/pkl/base/shared-content.pkl config/pkl/base/shared-content-automated.pkl config/pkl/renderers/metadata-coverage-check.pkl` passed; `git diff --check -- config/pkl/base/shared-content.pkl config/pkl/base/shared-content-automated.pkl config/pkl/base/shared-content-automated-code.pkl config/pkl/renderers/common.pkl config/pkl/renderers/opencode-metadata.pkl config/pkl/renderers/opencode-automated-metadata.pkl config/pkl/renderers/claude-metadata.pkl` passed; `rg -n "sce-context-generate-html|context/html/index.html" config/pkl/base config/pkl/renderers` found the expected aggregation, metadata, and output-contract references. Attempted `nix run .#pkl-check-generated`, but this environment has no `nix` executable (`zsh:1: command not found: nix`), so full Nix parity/check validation remains for a Nix-equipped session.
  - Context-sync classification: root-important for the overall generated skill surface because this task wires the new cross-target generated skill into renderer metadata. Durable current-state documentation remains scheduled for T05 after generated outputs are refreshed in T04; this task's post-implementation sync verified no root/domain context edits are required yet beyond the plan evidence because generated target files are not emitted until T04.

- [x] T04: `Regenerate generated skill outputs` (status:done)
  - Task ID: T04
  - Goal: Regenerate generated config artifacts so the new skill appears under generated skill directories.
  - Boundaries (in/out of scope): In - run the canonical generation command and review generated `SKILL.md` outputs for the new slug. Out - manual edits to generated outputs after regeneration except to correct canonical source and regenerate again.
  - Done when: `config/.opencode/skills/sce-context-generate-html/SKILL.md` exists with correct frontmatter/body, and any other generated target outputs expected by renderer wiring are present and deterministic.
  - Verification notes (commands or checks): `nix develop -c pkl eval -m . config/pkl/generate.pkl`; inspect generated skill files; `nix run .#pkl-check-generated` should report no generated drift after regeneration.
  - Completion evidence (2026-06-26): Attempted canonical `nix develop -c pkl eval -m . config/pkl/generate.pkl`, but this environment still has no `nix` executable (`zsh:1: command not found: nix`). Used host `pkl eval -m . config/pkl/generate.pkl` to regenerate generated-owned outputs from canonical Pkl sources. Confirmed generated skill files now exist at `config/.opencode/skills/sce-context-generate-html/SKILL.md`, `config/automated/.opencode/skills/sce-context-generate-html/SKILL.md`, and `config/.claude/skills/sce-context-generate-html/SKILL.md`; targeted `rg` verified expected slug, `context/html/index.html`, pre-generation sync, and diagram requirement text in all three outputs. Targeted checks passed: `pkl eval config/pkl/base/shared-content.pkl config/pkl/base/shared-content-automated.pkl config/pkl/renderers/metadata-coverage-check.pkl`; `git diff --check --` for the three generated skill files; repeat `pkl eval -m . config/pkl/generate.pkl` left only the three expected untracked generated skill directories; `IN_NIX_SHELL=impure ./config/pkl/check-generated.sh` reported `Generated outputs are up to date.`. `nix run .#pkl-check-generated` remains blocked by missing `nix` in this environment.
  - Context-sync classification: root-important for the overall generated skill surface, but this T04 slice only materializes previously authored/wired generated outputs. Post-implementation context sync verified the root shared files (`context/overview.md`, `context/architecture.md`, `context/glossary.md`, `context/patterns.md`, `context/context-map.md`) are under 250 lines and still describe the generated-config architecture accurately; focused `rg` showed the new skill/output convention is not yet present in durable context outside this active plan, which matches the explicit T05 scope to document the current-state skill surface. No durable root/domain context edits were made in T04 to avoid executing T05 early.

- [x] T05: `Document current-state skill surface` (status:done)
  - Task ID: T05
  - Goal: Update durable context so future sessions can discover the new HTML-doc generation skill and its output location.
  - Boundaries (in/out of scope): In - update focused context files such as `context/context-map.md`, `context/overview.md`, `context/architecture.md`, `context/patterns.md`, or a new focused domain file only if warranted by `sce-context-sync` classification. Out - lengthy changelog-style summaries or duplicating the full skill body in context.
  - Done when: Context reflects the current generated skill surface and `context/html/` output convention where appropriate, with discoverability links updated.
  - Verification notes (commands or checks): Follow `sce-context-sync`; confirm edited context files remain current-state oriented and under the repo's context line-length guidance.
  - Completion evidence (2026-06-26): Added focused current-state documentation at `context/sce/context-html-generation-skill.md` covering the generated `sce-context-generate-html` skill surface, generated target paths, canonical Pkl ownership, required pre-generation `sce-context-sync`, and `context/html/index.html` output convention. Updated discoverability/current-state references in `context/context-map.md`, `context/overview.md`, `context/architecture.md`, `context/patterns.md`, and `context/glossary.md` without duplicating the full skill body. Targeted verification passed: `git diff --check -- context/context-map.md context/overview.md context/architecture.md context/patterns.md context/glossary.md context/sce/context-html-generation-skill.md context/plans/sce-context-generate-html-skill.md`; targeted `rg` confirmed `sce-context-generate-html`, `context/html/index.html`, and the focused domain file links are discoverable; `wc -l` reported edited context files under 250 lines (`context/sce/context-html-generation-skill.md` 26, `context/context-map.md` 81, `context/overview.md` 122, `context/architecture.md` 160, `context/patterns.md` 186, `context/glossary.md` 207). `IN_NIX_SHELL=impure ./config/pkl/check-generated.sh` reported generated outputs are up to date. Attempted `nix run .#pkl-check-generated`, but this environment has no `nix` executable (`zsh:1: command not found: nix`), so Nix-backed parity/check validation remains for T06 or a Nix-equipped session.
  - Context-sync classification: root-important. This task documents a generated cross-target skill surface and the new `context/html/` documentation convention, so focused root/domain context edits were required and completed.

- [x] T06: `Validate generated skill and cleanup` (status:done)
  - Task ID: T06
  - Goal: Perform final validation and cleanup for the plan.
  - Boundaries (in/out of scope): In - run repository validation, generated parity checks, confirm generated outputs and context are clean, remove temporary artifacts. Out - additional feature expansion beyond the new skill.
  - Done when: `nix run .#pkl-check-generated` and `nix flake check` pass or any failures are documented with clear blockers; generated outputs are deterministic; no temporary files remain outside approved `context/tmp/` usage; the plan records validation evidence.
  - Verification notes (commands or checks): `nix run .#pkl-check-generated`; `nix flake check`; final `sce-context-sync` verification that feature documentation exists and is linked.
  - Completion evidence (2026-06-28): Required Nix-backed final checks were attempted but blocked in this environment because `nix` is not installed: `nix run .#pkl-check-generated` exited 127 with `zsh:1: command not found: nix`, and `nix flake check` exited 127 with the same diagnostic. Host fallback validation passed: `IN_NIX_SHELL=impure ./config/pkl/check-generated.sh` reported `Generated outputs are up to date.`; `pkl eval config/pkl/base/shared-content.pkl config/pkl/base/shared-content-automated.pkl config/pkl/renderers/metadata-coverage-check.pkl` passed; targeted `rg` confirmed `sce-context-generate-html`, required `sce-context-sync`, `context/html/index.html`, and diagram requirements in the generated OpenCode manual, OpenCode automated, and Claude skill outputs plus linked context documentation; targeted `git diff --check -- ...` passed for the plan, generated skill outputs, and relevant context files. A targeted temporary-artifact scan excluding `.git`, `context/tmp`, and `node_modules` found no `*~`, `*.tmp`, `.DS_Store`, or `*.bak` cleanup candidates. Existing unrelated root `.opencode/**` worktree changes were observed and left untouched. Final context-sync verification confirmed `context/sce/context-html-generation-skill.md` exists and is linked from `context/context-map.md`, with no additional context drift found for this validation-only task.
  - Context-sync classification: verify-only. This final validation task did not change the generated skill behavior or documentation contract beyond recording validation evidence in the active plan; durable current-state context already represents the completed skill surface and output convention.

## Validation Report

### Commands run

- `nix run .#pkl-check-generated` -> exit 127 (`zsh:1: command not found: nix`)
- `nix flake check` -> exit 127 (`zsh:1: command not found: nix`)
- `IN_NIX_SHELL=impure ./config/pkl/check-generated.sh` -> exit 0 (`Generated outputs are up to date.`)
- `pkl eval config/pkl/base/shared-content.pkl config/pkl/base/shared-content-automated.pkl config/pkl/renderers/metadata-coverage-check.pkl` -> exit 0
- `git diff --check -- ...` for the plan, generated skill outputs, and relevant context files -> exit 0
- Targeted `rg` checks for `sce-context-generate-html`, `sce-context-sync`, `context/html/index.html`, and diagram requirements -> exit 0
- Targeted temporary-artifact scan excluding `.git`, `context/tmp`, and `node_modules` -> exit 0 with no cleanup candidates

### Failed checks and follow-ups

- Required Nix checks could not run because this environment has no `nix` executable. Re-run `nix run .#pkl-check-generated` and `nix flake check` in a Nix-equipped environment before release/sign-off that requires Nix-backed validation.

### Success-criteria verification

- [x] Generated skill exists at `config/.opencode/skills/sce-context-generate-html/SKILL.md` and equivalent automated OpenCode / Claude generated targets.
- [x] Skill body requires `sce-context-sync` before HTML generation.
- [x] Skill body defines `context/html/index.html` as the deterministic browser entrypoint.
- [x] Skill body requires overview-derived content, local/inline CSS, helpful diagrams, and browser-visible diagram rendering checks.
- [x] Canonical Pkl sources and metadata evaluate successfully with host Pkl.
- [x] Generated-output parity fallback reports no drift.
- [x] Durable context documents and links the generated skill surface through `context/sce/context-html-generation-skill.md` and `context/context-map.md`.

### Residual risks

- Nix-backed parity/full flake validation remains unexecuted in this environment due to missing `nix`; fallback checks indicate generated outputs are deterministic and context is aligned.

## Open questions

- None blocking. The plan treats `sce-context-generate-html` as the canonical skill slug and `context/html/index.html` as the default generated documentation entrypoint based on the user's requested name and `context/` HTML output location.
