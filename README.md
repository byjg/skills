# ByJG Agent Skills

Skills that teach a coding agent (Claude Code, Codex, Cursor, ...) how to work with the
ByJG open source components. Documentation: [https://opensource.byjg.com/docs/ai/agent-skills](https://opensource.byjg.com/docs/ai/agent-skills).

> [!IMPORTANT]
> This repository is generated automatically. Each skill is maintained in the repository
> of the component it describes, in `.claude/skills/<name>/`, and published here on every
> change to that repository's `master` branch. Changes made here are overwritten by the
> next publish: open issues and pull requests in the component's repository instead.

## Skills

No skills are published yet.

## Install

**Claude Code**: add the catalog once, then install the skill:

```text
/plugin marketplace add byjg/skills
/plugin install <skill>@byjg
```

Or declare it in a project's `.claude/settings.json`, so Claude Code offers it to
everyone who opens the project:

```json
{
  "extraKnownMarketplaces": {
    "byjg": {
      "source": {
        "source": "github",
        "repo": "byjg/skills"
      }
    }
  },
  "enabledPlugins": {
    "<skill>@byjg": true
  }
}
```

**Other agents** that read Agent Skills (Codex, Cursor, Gemini CLI, ...):

```bash
npx skills add byjg/skills --skill <skill>
```
