# claude-skills

A personal library of [Claude Code](https://claude.com/claude-code) **skills** — reusable, model-invoked instruction sets that Claude loads on demand when a task matches. All of them are about the same thing: checking that a change is the *right* one before it ships.

Each skill is a folder under [`skills/`](skills/) containing a `SKILL.md` with YAML frontmatter (`name` + `description`) and a body of instructions. Claude reads the `description` to decide when a skill is relevant, then follows the body. None of them refers to a specific project: copy one in and it works.

## Skills

| Skill | Language | What it does |
|-------|----------|--------------|
| [`pre-commit-review`](skills/pre-commit-review/SKILL.md) | English | Pre-commit review that judges whether a diff is the **right** change — scope, contracts, simplicity, and intent-alignment — not just one that passes CI. |
| [`revue`](skills/revue/SKILL.md) | French | The French version of `pre-commit-review`: same steps, same verdict format. |
| [`pr-revue-smoke`](skills/pr-revue-smoke/SKILL.md) | French | PR workflow: open as draft, two read-only subagent reviews run one after the other (design, then code, so the second catches what the first missed or broke), one smoke-test subagent, and only then `gh pr ready`. Every comment the agent posts carries a header saying Claude writes on the user's behalf. A personal method, not a repo rule: install it at user scope. Assumes GitHub + `gh`. |

### `pre-commit-review` / `revue` — "is this the *right* change?"

CI proves the planned checks pass; it does not prove we checked the *right* things. This review judges the change itself before line-by-line reading:

- **Step 0 — Establish intent.** Reconstruct what the change was *supposed* to do (original request, issue, acceptance criteria) before judging the diff.
- **Step 1 — Collect the diff.** `git status`, `git diff`, and test state *before* judging.
- **Step 2 — Nine questions**, each answered with evidence from the diff (`file:line`), not an impression: Need, Scope, Contracts, Consistency, Tests, Security, Operability, Simplicity, Alignment.
- **Step 3 — Verdict**: `READY TO COMMIT` / `FIXES NEEDED` / `DISCUSSION REQUIRED`, in a fixed table format.

Pick one of the two: they are the same skill in two languages, and both trigger on the same requests.

## Install

### As plugins (recommended)

This repository is a Claude Code plugin marketplace; each skill is its own plugin, so you install only what you want. In Claude Code:

```
/plugin marketplace add BhuvanArn/claude-skills
/plugin install pre-commit-review@bhuvan-skills
```

Or from your shell:

```bash
claude plugin marketplace add BhuvanArn/claude-skills
claude plugin install pre-commit-review@bhuvan-skills
```

On Claude Code 2.1.275 or later, one line in a session adds the marketplace and installs a plugin:

```
/plugin install pre-commit-review --marketplace BhuvanArn/claude-skills
```

In every command above, swap `pre-commit-review` for `revue` or `pr-revue-smoke` to install another plugin.

Third-party marketplaces don't auto-update by default: turn it on in `/plugin` → **Marketplaces** → `bhuvan-skills` → **Enable auto-update**, or update by hand with `claude plugin update <plugin>@bhuvan-skills`.

A plugin skill is invoked with its plugin prefix, e.g. `/pre-commit-review:pre-commit-review`; Claude also loads it on its own when a request matches its description.

### By copying the folder

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

`pr-revue-smoke` belongs in `~/.claude/skills/`, not in a repository: committed there, it would become a rule imposed on every contributor.

## Repository layout

```
.claude-plugin/
  marketplace.json  # lists each skill as an installable plugin
skills/
  <name>/
    SKILL.md        # required: frontmatter (name, description) + instructions
    (references/)   # optional: supporting files the skill points to
```

The `description` is the most important line — it's what Claude matches against to decide relevance, so it should name the concrete triggers ("when the user says…", "before committing…").

To add a skill: create `skills/<name>/SKILL.md`, add an entry to `.claude-plugin/marketplace.json` (copy an existing one and change `name`, `skills`, `description`), then run `claude plugin validate .`.

## License

MIT — see [LICENSE](LICENSE).
