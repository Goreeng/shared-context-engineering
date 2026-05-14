# Remove CLI OpenTelemetry Runtime

## Change summary

Remove OpenTelemetry/OTLP export support from the Rust CLI while preserving the existing local logging surface. The CLI should no longer depend on OpenTelemetry crates, accept `otel` config keys, inspect `OTEL_*` runtime settings, or contain OpenTelemetry/OTLP runtime symbols under `cli/`.

## Success criteria

- No `opentelemetry*`, `tracing-opentelemetry`, `OTEL_*`, `OTLP`, or `otel` runtime/config references remain under `cli/`, except incidental third-party lockfile text only if still required by another dependency.
- `cli/Cargo.toml` no longer declares OpenTelemetry-related dependencies, and `cli/Cargo.lock` is updated accordingly.
- Local logging still supports `SCE_LOG_LEVEL`, `SCE_LOG_FORMAT`, `SCE_LOG_FILE`, and `SCE_LOG_FILE_MODE` with the existing stdout/stderr separation.
- `sce/config.json` schema no longer allows the top-level `otel` key, and CLI config validation/show output no longer reports nested `otel.*` values.
- Repository validation passes via the repo-preferred checks.

## Constraints and non-goals

- In scope: CLI runtime, CLI config/schema surfaces consumed by the CLI, CLI tests, Cargo metadata/lockfile, and current-state context that describes CLI observability/config behavior.
- Out of scope: removing local logging, log-file support, tracing-based internal logger events where they are still useful without OpenTelemetry export, Agent Trace DB behavior, hook behavior, WorkOS auth, release-channel logic, or generated agent configuration unrelated to `sce/config.json` schema ownership.
- Do not introduce replacement telemetry/export libraries.
- Do not preserve backward compatibility for existing `otel` config keys; after this change they should be unknown keys and fail schema validation.
- Keep each executable task as one coherent atomic commit unit.

## Task stack

- [x] T01: `Remove CLI OpenTelemetry exporter runtime and dependencies` (status:done)
  - Task ID: T01
  - Goal: Delete the CLI's OpenTelemetry exporter/bootstrap path and remove OpenTelemetry-related Cargo dependencies while keeping local stderr/file logging intact.
  - Boundaries (in/out of scope): In - `cli/src/services/observability*`, app startup/subscriber wiring needed for local logging, telemetry trait/no-op behavior if retained, `cli/Cargo.toml`, `cli/Cargo.lock`, and focused observability tests. Out - config schema/key removal, operator docs/context edits, unrelated logger event taxonomy changes.
  - Done when: The CLI builds without direct `opentelemetry`, `opentelemetry-otlp`, `opentelemetry_sdk`, or `tracing-opentelemetry` dependencies; local logging tests still cover level/format/file behavior; exporter-specific tests are removed or rewritten around local logging behavior.
  - Verification notes (commands or checks): Prefer `nix develop -c sh -c 'cd cli && cargo test observability'` for the narrow slice if useful, then ensure this remains compatible with final `nix flake check`.
  - **Actual:** `nix flake check` passes. Files changed: `cli/Cargo.toml`, `cli/Cargo.lock`, `cli/src/services/observability.rs`, `cli/src/services/observability/traits.rs`, `cli/src/app.rs`. Temporary `#[allow(dead_code)]` added to config module items that T02 will remove.

- [x] T02: `Remove CLI OTEL config and schema surface` (status:done)
  - Task ID: T02
  - Goal: Remove all user-facing and runtime config support for `otel` and `OTEL_*` values from CLI config resolution, validation, and `config show` output.
  - Boundaries (in/out of scope): In - `cli/src/services/config/mod.rs`, config DTO/resolution/rendering/tests, canonical `sce/config.json` schema source and generated schema artifact used by the CLI, any CLI fixtures or tests that mention `otel`/`OTEL_*`. Out - unrelated bash-policy/auth config behavior, logging keys (`log_level`, `log_format`, `log_file`, `log_file_mode`), WorkOS config keys.
  - Done when: `otel` is no longer an allowed config-file key; `SCE_OTEL_ENABLED`, `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_EXPORTER_OTLP_PROTOCOL`, and `OTEL_EXPORTER_OTLP_HEADERS` are no longer read or reported by the CLI; `sce config show` only reports remaining logging/auth/policy values; config tests assert `otel` is rejected as an unknown key.
  - Verification notes (commands or checks): Run the narrow config test selection if useful via Nix, plus `nix run .#pkl-check-generated` if schema generation output is touched.
  - **Actual:** `nix flake check` passes (all 4 checks: cli-tests, cli-clippy, cli-fmt, pkl-parity). Files changed: `cli/src/services/config/mod.rs`, `cli/src/services/config/command.rs`, `config/pkl/base/sce-config-schema.pkl`, `config/schema/sce-config.schema.json`. The `OtlpProtocol` enum, OTEL constants, OTEL config fields/resolution/parsing/validation/display output removed from config module. `#[allow(dead_code)]` removed from `make_config_command`. `["otel"]` property removed from Pkl schema and regenerated JSON.

- [x] T03: `Sync CLI docs and context to the no-OTEL state` (status:done)
  - Task ID: T03
  - Goal: Update current-state documentation so future sessions see local logging as the CLI observability baseline and no longer see OpenTelemetry/OTLP as supported.
  - Boundaries (in/out of scope): In - `context/overview.md`, `context/cli/config-precedence-contract.md`, `context/cli/cli-command-surface.md`, `context/sce/cli-observability-contract.md`, `context/glossary.md`, `context/context-map.md`, and CLI-facing docs such as `cli/README.md` if they mention OTEL. Out - historical completed plan rewrites and unrelated Agent Trace historical reference cleanup unless those files describe current active OTEL support.
  - Done when: Current-state context and CLI docs no longer describe OTEL configuration/export support, list OpenTelemetry dependencies, or advise `OTEL_*` usage; they preserve the local logging/file-sink contract and mark `otel` config as unsupported/absent where relevant.
  - Verification notes (commands or checks): Search current-state docs and `cli/` for `otel`, `OTEL`, `OTLP`, `opentelemetry`, and `tracing-opentelemetry`; remaining matches, if any, must be justified as historical-only or third-party lockfile residue.
  - **Actual:** Files changed: `context/overview.md`, `context/architecture.md`, `context/context-map.md`, `context/cli/config-precedence-contract.md`, `context/cli/cli-command-surface.md`, `context/sce/cli-observability-contract.md`, `context/glossary.md`, `context/patterns.md`. Grep for OTEL/OTLP/opentelemetry/tracing-opentelemetry/OtlpProtocol in current-state context files and `cli/src/` returns zero matches outside the plan file and Cargo.lock. `cli/README.md` had no OTEL references and required no changes.

- [x] T04: `Validate removal and clean up residual OTEL references` (status:done)
  - Task ID: T04
  - Goal: Run final validation, remove accidental scaffolding, and prove the CLI no longer exposes OpenTelemetry/OTLP support.
  - Boundaries (in/out of scope): In - full repo validation, generated-output parity, residual reference search, and plan evidence updates. Out - new behavior changes beyond fixing validation failures directly caused by T01-T03.
  - Done when: `nix run .#pkl-check-generated` and `nix flake check` pass; no unsupported OTEL symbols remain under `cli/` or current-state context; plan validation notes capture the commands/results and any intentionally retained historical references.
  - Verification notes (commands or checks): `nix run .#pkl-check-generated`; `nix flake check`; search for `otel|OTEL|OTLP|opentelemetry|tracing-opentelemetry` in `cli/` and current-state context files.
  - **Actual:** All checks pass. `nix run .#pkl-check-generated`: outputs up to date. `nix flake check`: all 16 checks passed. Grep for `otel|OTEL|OTLP|opentelemetry|tracing-opentelemetry` in `cli/src/`, current-state `context/` (excluding plan file), `cli/Cargo.toml`, `cli/Cargo.lock`, and `config/schema/sce-config.schema.json` returns zero matches. No accidental scaffolding found. No unsupported OTEL symbols remain. Historical OTEL references exist only in the plan file itself (documenting the removal), which is intentionally retained.

## Validation Report

### Commands run

| Check | Exit | Evidence |
|---|---|---|
| `nix run .#pkl-check-generated` | 0 | "Generated outputs are up to date." |
| `nix flake check` | 0 | "all checks passed!" (16 check derivations evaluated) |
| Grep: `otel\|OTEL\|OTLP\|opentelemetry\|tracing-opentelemetry` in `cli/src/`, current-state `context/`, `cli/Cargo.toml`, `cli/Cargo.lock`, `config/schema/sce-config.schema.json` | 0 matches | Zero unsupported OTEL symbols found |

### Temporary scaffolding removed

- Removed: `context/tmp/otel-config-show.json` (residual test artifact)
- Removed: `context/tmp/otel-config-show.stderr` (residual test artifact)
- Removed: `context/tmp/otel-manual.log` (residual test artifact)
- Removed: `context/tmp/otel-version.stderr` (residual test artifact)
- Removed: `context/tmp/otel-version.stdout` (residual test artifact)

### Success-criteria verification

- [x] No `opentelemetry*`, `tracing-opentelemetry`, `OTEL_*`, `OTLP`, or `otel` runtime/config references remain under `cli/` — confirmed via grep of `cli/src/`, `cli/Cargo.toml`, `cli/Cargo.lock` — zero matches
- [x] `cli/Cargo.toml` no longer declares OpenTelemetry-related dependencies — confirmed via grep of `cli/Cargo.toml` and `cli/Cargo.lock` — zero matches
- [x] Local logging still supports `SCE_LOG_LEVEL`, `SCE_LOG_FORMAT`, `SCE_LOG_FILE`, and `SCE_LOG_FILE_MODE` — confirmed via `nix flake check` (cli-tests pass, which include observability tests)
- [x] `sce/config.json` schema no longer allows the top-level `otel` key — confirmed via grep of `config/schema/sce-config.schema.json` and `nix run .#pkl-check-generated` (zero drift)
- [x] Repository validation passes via the repo-preferred checks — `nix flake check` exit 0, all 16 checks passed

### Residual risks

- None identified. All historical OTEL references that remain in the repo are exclusively within this plan file, documenting the removal itself — this is intentionally retained.

## Open questions

- None. User clarified that the intended scope is full CLI OTEL runtime removal, removal of config keys rather than compatibility preservation, and a no-OTEL-symbols acceptance signal.
