# Remove integrations/install

## Change summary

Remove the standalone `integrations/install/` Rust install-channel integration runner from the repository and eliminate all active repo wiring that references it.

## Success criteria

- `integrations/install/` is deleted from the repository.
- Root flake no longer defines `integrationsInstall*` source/package/artifact/check derivations.
- Root flake no longer exposes the `install-channel-integration-tests` app unless it is replaced by an explicitly approved non-`integrations/install` implementation.
- Dependabot no longer references `/integrations/install`.
- Durable context no longer describes the removed runner, its dedicated checks, or its opt-in app as current state.
- Repository validation passes after the removal.

## Constraints and non-goals

- Do not preserve the standalone Rust runner under a new path.
- Do not add a replacement install-channel integration harness unless separately requested.
- Do not change primary `sce` CLI behavior, npm/Cargo/Flatpak distribution implementation, or generated OpenCode/Claude assets beyond any required documentation/context sync.
- Prefer removing stale references over leaving compatibility shims for the removed flake app.

## Task stack

- [x] T01: `Remove integrations/install code and repo wiring` (status:done)
  - Task ID: T01
  - Goal: Delete the standalone install-channel integration runner and remove all active Nix/GitHub configuration that points at it.
  - Boundaries (in/out of scope): In - delete `integrations/install/`; remove `integrationsInstallSrc`, `integrationsInstallCargoArgs`, `integrationsInstallCargoArtifacts`, `integrationsInstallPackage`, `installChannelIntegrationTestsApp`, the `apps.install-channel-integration-tests` entry, the `integrations-install-*` checks, and the Dependabot `/integrations/install` Cargo entry. Out - adding a new runner, changing release packaging, changing `sce` CLI runtime behavior.
  - Done when: `integrations/install/` no longer exists; `flake.nix` has no references to `integrations/install`, `integrationsInstall`, `integrations-install`, or `install-channel-integration-tests`; `.github/dependabot.yml` has no `/integrations/install` entry.
  - Verification notes (commands or checks): Search the repository for `integrations/install`, `integrationsInstall`, `integrations-install`, and `install-channel-integration-tests`; run `nix flake check` if feasible after the edit.
  - Completed: 2026-07-02
  - Files changed: deleted `integrations/install/`; updated `flake.nix`; updated `.github/dependabot.yml`.
  - Evidence: `git grep -n -E "integrations/install|integrationsInstall|integrations-install|install-channel-integration-tests" -- flake.nix .github/dependabot.yml || true` returned no matches; `nix flake check` passed twice (`all checks passed!`). `nix run .#pkl-check-generated` failed on pre-existing generated OpenCode plugin formatting drift outside this task's scope.
  - Notes: Implementation classified as an important context-sync change because active root-flake checks/apps and the standalone runner were removed; `sce-context-sync` updated current-state docs during the T01 done gate without marking T02 complete.

- [x] T02: `Sync current-state context for removed runner` (status:done)
  - Task ID: T02
  - Goal: Update durable context so future sessions do not treat the removed integration runner, app, or checks as current state.
  - Boundaries (in/out of scope): In - update `context/overview.md`, `context/architecture.md`, `context/glossary.md`, `context/context-map.md`, and `context/sce/optional-install-channel-integration-test-entrypoint.md` as needed to remove or reclassify the deleted install-channel runner surface. Out - broad prose rewrites, historical migration narratives, or unrelated distribution-contract edits.
  - Done when: Context files no longer list `integrations/install`, `integrations-install-*` checks, or `apps.install-channel-integration-tests` as active current state; if the optional install-channel context file becomes obsolete, it is either removed or rewritten as removed/deferred state and unlinked from `context/context-map.md`.
  - Verification notes (commands or checks): Search `context/` for `integrations/install`, `integrations-install`, and `install-channel-integration-tests`; confirm remaining mentions, if any, are explicitly historical/removed and intentional.
  - Completed: 2026-07-02
  - Files changed: `context/overview.md`, `context/architecture.md`, `context/glossary.md`, `context/patterns.md`, `context/context-map.md`, deleted `context/sce/optional-install-channel-integration-test-entrypoint.md`, and updated this plan file.
  - Evidence: `git grep -n -E "integrations/install|integrations-install|integrationsInstall|install-channel-integration-tests" -- context ':!context/plans' || true` returns one non-plan match in `context/overview.md` that explicitly states the former runner and flake app are not active current-state surfaces; `test ! -e context/sce/optional-install-channel-integration-test-entrypoint.md` passed.
  - Notes: T02 required no additional durable context edits beyond the sync already performed during T01's done gate; this task records verification and completion.

- [x] T03: `Validation and cleanup` (status:done)
  - Task ID: T03
  - Goal: Validate the repository after deletion and ensure no stale artifacts or references remain.
  - Boundaries (in/out of scope): In - run full repo validation, generated-output parity, final reference search, and remove any temporary files created during the task. Out - implementing replacement integration tests or unrelated formatting churn.
  - Done when: `nix flake check` and `nix run .#pkl-check-generated` pass, or any failures are captured with clear blocker notes; no stale references to the removed path/surface remain outside intentional historical notes.
  - Verification notes (commands or checks): `nix flake check`; `nix run .#pkl-check-generated`; final repository search for `integrations/install`, `integrations-install`, `integrationsInstall`, and `install-channel-integration-tests`.
  - Completed: 2026-07-02
  - Files changed: `context/plans/remove-integrations-install.md`.
  - Evidence: `nix flake check` passed (`all checks passed!`). `nix run .#pkl-check-generated` failed with generated output drift limited to existing OpenCode plugin line-wrapping differences in `config/.opencode/plugins/{sce-agent-trace.ts,sce-bash-policy.ts}` and `config/automated/.opencode/plugins/{sce-agent-trace.ts,sce-bash-policy.ts}`; this drift is unrelated to removal of `integrations/install`. Final search for `integrations/install`, `integrations-install`, `integrationsInstall`, and `install-channel-integration-tests` found only plan-local mentions plus one intentional current-state note in `context/overview.md` stating the former runner/app are not active; `integrations/install/**` has no files.
  - Notes: No temporary task files were created. Context sync was verify-only for this validation task; root/domain context already represents the removed runner/app/checks as non-active current state and no additional durable context edits were needed.

## Validation Report

### Commands run

- `nix flake check` -> exit 0; key output: `all checks passed!`.
- `nix run .#pkl-check-generated` -> exit non-zero; key output: generated output drift detected only in OpenCode plugin line wrapping for `config/.opencode/plugins/{sce-agent-trace.ts,sce-bash-policy.ts}` and `config/automated/.opencode/plugins/{sce-agent-trace.ts,sce-bash-policy.ts}`. This was previously identified during T01 and is unrelated to `integrations/install` removal.
- Final reference search for `integrations/install`, `integrations-install`, `integrationsInstall`, and `install-channel-integration-tests` -> only plan-local mentions plus one intentional removed-state note in `context/overview.md`.
- `integrations/install/**` file existence check -> no files found.

### Success-criteria verification

- [x] `integrations/install/` is deleted from the repository -> confirmed by no files under `integrations/install/**`.
- [x] Root flake no longer defines `integrationsInstall*` source/package/artifact/check derivations -> confirmed by final reference search and `flake.nix` diff.
- [x] Root flake no longer exposes `install-channel-integration-tests` -> confirmed by final reference search and `flake.nix` diff.
- [x] Dependabot no longer references `/integrations/install` -> confirmed by `.github/dependabot.yml` diff.
- [x] Durable context no longer describes the removed runner/app/checks as current state -> confirmed by context sync; remaining non-plan mention states the former runner/app are not active current-state surfaces.
- [~] Repository validation passes after removal -> `nix flake check` passed; `nix run .#pkl-check-generated` still fails on unrelated generated OpenCode plugin formatting drift.

### Failed checks and follow-ups

- `nix run .#pkl-check-generated` remains blocked by unrelated generated OpenCode plugin line-wrapping drift. Follow-up should regenerate or normalize generated OpenCode plugin outputs in a separate scoped task/plan.

### Residual risks

- Generated-output parity remains red until the unrelated OpenCode plugin formatting drift is resolved.

## Open questions

- None.
