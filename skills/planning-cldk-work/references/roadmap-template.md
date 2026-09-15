# The roadmap artifact

**Template:** `codellm-devkit/.github` → [`docs/design/roadmap-template.md`](https://github.com/codellm-devkit/.github/blob/main/docs/design/roadmap-template.md).
Copy it; do not re-derive the sections here. Same rule as issue bodies — the org repo
owns the shape, this skill owns when and why.

**Lands at:** `codellm-devkit/.github` → `docs/design/roadmap.md`, committed, beside the parent issues and
cross-repo specs it coordinates.

## Skill-side rules

- **One roadmap, amended in place.** Never a second file per quarter — git history is the record of
  how thinking changed, which is what makes the doc worth more than an issue.
- **Rows are contract decisions, not themes.** A theme decomposes into several rows before the
  collision sweep can compare anything. This is the gate, not a style preference.
- **`Starting now` holds exactly one candidate.** Everything else on the roadmap has no issue yet,
  by design — filing ahead is what turns a plan into inventory.
- **Dropped candidates move to `Not now` with a reason**, rather than being deleted. That a thing
  was considered and rejected is often the most useful fact the roadmap carries.
