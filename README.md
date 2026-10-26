# ByJG Agent Skills

Skills that teach a coding agent (Claude Code, Codex, Cursor, ...) how to work with the
ByJG open source components. Documentation: [https://opensource.byjg.com/docs/ai/agent-skills](https://opensource.byjg.com/docs/ai/agent-skills).

> [!IMPORTANT]
> This repository is generated automatically. Each skill is maintained in the repository
> of the component it describes, in `.claude/skills/<name>/`, and published here on every
> change to that repository's `master` branch. Changes made here are overwritten by the
> next publish: open issues and pull requests in the component's repository instead.

## Skills

| Skill | Description | Source |
|---|---|---|
| [gluo-developer](skills/gluo-developer) | Expert guide for working on a Gluo project: a PHP REST API created from byjg/php-gluo (composer create-project byjg/gluo) on the byjg/gluo-core framework. Use this skill whenever the user is working on such a project — including setting up the dev environment, creating new CRUD features, scaffolding with the code generator, modifying or fixing existing features, writing or updating tests, running migrations, and maintaining OpenAPI documentation. Trigger this skill for any task related to models, repositories, services, REST controllers, attributes, authentication, DI configuration, or the byjg framework stack in this codebase. Even if the user says something generic like "add a new endpoint" or "fix this bug", use this skill if they're working in this project. | [byjg/php-gluo](https://github.com/byjg/php-gluo/tree/master/.claude/skills/gluo-developer) |

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
