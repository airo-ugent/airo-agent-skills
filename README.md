# airo-agent-skills

A collection of reusable skills for coding agents, maintained by the AIRO lab.

A skill is a folder with a `SKILL.md` that an agent loads when it becomes relevant. The point of
keeping them here rather than in each person's home directory is that a convention only holds if
everyone's agent knows about it.

| Skill | What it does |
|---|---|
| [`airo-documentation-writer`](skills/airo-documentation-writer) | Writes and revises documentation in the AIRO house voice, distilled from `airo-mono` |

This repo is also a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces),
which is how the skills get installed. The `skills/` layout is the portable part; the two files in
`.claude-plugin/` are the Claude Code packaging around it.

Code style (Black, isort, Flake8) stays in each repo's `.pre-commit-config.yaml` — that is enforced
on commit and needs no agent. This repo is for the conventions a formatter cannot check.

## Installation

```shell
/plugin marketplace add airo-ugent/airo-agent-skills
/plugin install airo-agent-skills@airo-agent-skills
/reload-plugins
```

`/reload-plugins` activates the skills in the session you already have open; without it you get
them next session. A skill fires on its own when its description matches what you asked for, and
can be invoked explicitly as `/airo-agent-skills:airo-documentation-writer`.

To pick up skills and fixes that others have pushed since:

```shell
/plugin marketplace update airo-agent-skills
```

### For everyone on a project

Commit this to a repo's `.claude/settings.json` and anyone who opens that repo is prompted to
install the marketplace, so a new student gets the lab's conventions without having to be told
they exist:

```json
{
  "extraKnownMarketplaces": {
    "airo-agent-skills": {
      "source": {
        "source": "github",
        "repo": "airo-ugent/airo-agent-skills"
      }
    }
  },
  "enabledPlugins": {
    "airo-agent-skills@airo-agent-skills": true
  }
}
```

## Adding a skill

```
skills/
    your-skill-name/
        SKILL.md            # required: frontmatter + the instructions
        references/         # optional: detail the agent reads only when it needs it
```

- **The directory name, the frontmatter `name`, and the invocation all have to agree.** Keep them
  identical.
- **The `description` is the trigger, and it is the only part loaded into every session.** Write it
  as the conditions under which the skill should fire, not as a summary of it, and name the words a
  user would actually type. `claude plugin details airo-agent-skills` prints what the collection
  costs per session — roughly 200 tokens per skill for the descriptions, and the body only once it
  fires. That budget is why detail belongs in `references/` rather than in `SKILL.md`.
- **Write the body as instructions to an agent, not as documentation for a human.** Concrete rules
  with real examples beat prose about principles.

## Validating

Run both before pushing. `--strict` promotes warnings to errors, which is what you want here:

```bash
claude plugin validate . --strict                       # the marketplace catalog
claude plugin validate .claude-plugin/plugin.json --strict   # the plugin, and every skill in it
claude plugin details airo-agent-skills                 # what was actually discovered, and the token cost
```

Validating is not optional busywork. A `description:` containing an unquoted `: ` is valid-looking
markdown and broken YAML — **the frontmatter is dropped silently and the skill loads with no
metadata**, which means it never triggers automatically and nothing tells you. Quote the value or
avoid the colon.

## Working on a skill

Symlink your checkout into your personal skills directory. It loads as
`airo-agent-skills@skills-dir` every session, straight from the working tree, so an edit is live
next session with no reinstall:

```bash
ln -s ~/projects/airo-agent-skills ~/.claude/skills/airo-agent-skills
```

Symlink or install, never both — two copies of a skill load as two separate skills, and you will
not know which one answered.
