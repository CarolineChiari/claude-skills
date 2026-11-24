# Claude Skills

A collection of skills for Claude, each packaged as a plugin. The repo is also a plugin marketplace, so you can install the skills straight from GitHub in Claude Code.

## Skills

| Skill | What it does |
| --- | --- |
| [edit-markup](plugins/edit-markup/skills/edit-markup/SKILL.md) | Teacher-style markup and clean rewrites of your own writing (Quora answers, Substack drafts, essays, emails). It produces a ruled-paper HTML page with red cuts, green inserts, numbered margin notes explaining each change, and a clean final version with a copy button. |

## Install

### Claude Code

Add this repo as a marketplace, then install a skill from it:

```
/plugin marketplace add CarolineChiari/claude-skills
/plugin install edit-markup@claude-skills
```

From a terminal, the same thing is:

```bash
claude plugin marketplace add CarolineChiari/claude-skills
```

```bash
claude plugin install edit-markup@claude-skills
```

### Claude.ai and the Claude app

1. Download the skill's folder (for example `plugins/edit-markup/skills/edit-markup/`).
2. Zip the folder so that `SKILL.md` sits inside a folder named after the skill.
3. Upload the zip in the Skills section of Claude's settings.

## Layout

```
.claude-plugin/marketplace.json        # lists every plugin in this repo
plugins/
  <plugin-name>/
    .claude-plugin/plugin.json         # plugin manifest
    skills/<skill-name>/SKILL.md       # the skill itself
```

## Adding a skill

1. Create `plugins/<name>/` with a `.claude-plugin/plugin.json` and `skills/<name>/SKILL.md`.
2. Add an entry for it to `.claude-plugin/marketplace.json`.
3. Check it with `claude plugin validate --strict .`
