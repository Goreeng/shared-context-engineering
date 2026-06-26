# Context HTML generation skill

Current generated skill surface:

- Skill slug: `sce-context-generate-html`
- Generated targets:
  - `config/.opencode/skills/sce-context-generate-html/SKILL.md`
  - `config/automated/.opencode/skills/sce-context-generate-html/SKILL.md`
  - `config/.claude/skills/sce-context-generate-html/SKILL.md`
- Canonical source ownership:
  - manual body: `config/pkl/base/shared-content-code.pkl`
  - automated body: `config/pkl/base/shared-content-automated-code.pkl`
  - aggregation: `config/pkl/base/shared-content.pkl`, `config/pkl/base/shared-content-automated.pkl`
  - metadata: `config/pkl/renderers/{common,opencode-metadata,opencode-automated-metadata,claude-metadata}.pkl`

Behavior contract:

- The skill generates static, human-readable HTML documentation from refreshed `context/` files.
- It must load and run `sce-context-sync` before writing HTML.
- Refreshed current-state context files are the source of truth; the skill must not invent behavior absent from context or verified code.
- Output is deterministic static documentation under `context/html/`.
- `context/html/index.html` is the default browser entrypoint.
- CSS must be inline or local to `context/html/`.
- Diagrams should be included when helpful and must render visibly in browser-compatible HTML rather than remaining raw unsupported Mermaid/code-fence text.

Generation remains instruction-only: no Rust CLI command, new dependency, or opaque build pipeline is part of this surface.
