# claude-skills

A personal library of [Claude Code](https://claude.com/claude-code) **skills** — reusable, model-invoked instruction sets that Claude loads on demand when a task matches.

Each skill is a folder under [`skills/`](skills/) containing a `SKILL.md` with YAML frontmatter (`name` + `description`) and a body of instructions. Claude reads the `description` to decide when a skill is relevant, then follows the body.

## Skills

| Skill | What it does |
|-------|--------------|
| [`pre-commit-review`](skills/pre-commit-review/SKILL.md) | **Project-agnostic** (English) pre-commit review that judges whether a diff is the **right** change — scope, contracts, simplicity, and intent-alignment — not just one that passes CI. Drop into any repo. |
| [`revue`](skills/revue/SKILL.md) | A French fine-tune of `pre-commit-review` for a Unity/C# project. Same structure, project-specific references. |

### `pre-commit-review` — the portable version

Start here for any project. It's the generalized, English form of `revue`: same intent → diff → 9 questions → verdict structure, with no references to any specific codebase. Nothing to swap — copy it in and it works.

### `revue` — "is this the *right* change?"

A French fine-tune of `pre-commit-review`, adapted for a Unity/C# game project. CI proves the planned checks pass; it does not prove we checked the *right* things. `revue` fills that gap by judging the change itself before line-by-line reading:

- **Step 0 — Establish intent.** Reconstruct what the change was *supposed* to do (original request, roadmap milestone, acceptance criteria) before judging the diff.
- **Step 1 — Collect the diff.** `git status`, `git diff`, and test state *before* judging.
- **Step 2 — Nine questions**, each answered with evidence from the diff (`file:line`), not an impression: Need, Scope, Contracts, Consistency, Tests, Security, Operability, Simplicity, Alignment.
- **Step 3 — Verdict**: `READY TO COMMIT` / `FIXES NEEDED` / `DISCUSSION REQUIRED`, in a fixed table format.

> **Note:** `revue` is tailored to one Unity/C# project — it references that project's test command (`dotnet test Simulation.Tests`), its architecture guardrails (the simulation package must never depend on Unity), and its design docs. The *structure* (intent → diff → 9 questions → verdict) is general and portable; the project-specific references would need swapping for another codebase.

## Using a skill

**Per project** — copy the skill folder into the project's `.claude/skills/`:

```bash
mkdir -p your-project/.claude/skills
cp -r skills/pre-commit-review your-project/.claude/skills/
```

**Globally (all projects)** — copy it into your user skills directory:

```bash
mkdir -p ~/.claude/skills
cp -r skills/pre-commit-review ~/.claude/skills/
```

Claude Code discovers the skill automatically. It triggers on the phrases in the skill's `description`, or you can invoke it explicitly (e.g. `/revue`).

## Anatomy of a skill

```
skills/
  <name>/
    SKILL.md        # required: frontmatter (name, description) + instructions
    (references/)   # optional: supporting files the skill points to
```

The `description` is the most important line — it's what Claude matches against to decide relevance, so it should name the concrete triggers ("when the user says…", "before committing…").

## License

MIT — see [LICENSE](LICENSE).
