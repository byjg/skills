# ByJG Agent Skills

[![Sponsor](https://img.shields.io/badge/Sponsor-%23ea4aaa?logo=githubsponsors&logoColor=white&labelColor=0d1117)](https://github.com/sponsors/byjg)
[![Opensource ByJG](https://img.shields.io/badge/opensource-byjg-success.svg)](https://opensource.byjg.com)
[![GitHub source](https://img.shields.io/badge/Github-source-informational?logo=github)](https://github.com/byjg/skills/)

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
| [feature-workflow](skills/feature-workflow) | Guided workflow for adding new features or fixing bugs in DockNimbus. Use this skill whenever the user asks to add a feature, fix a bug, implement a new API endpoint, add a CLI command, modify the UI, change agent behavior, or any task that involves changing functionality across the codebase layers (types, SDK, API handlers, repository, CLI, UI, agent). Trigger on phrases like 'add feature', 'fix bug', 'implement X', 'new command', 'new endpoint', 'agent not working', 'UI missing X', or any request that touches application logic. Even if the user mentions only one layer (e.g. 'fix the agent'), use this skill because the root cause often spans multiple layers. | [docknimbus/nimbus](https://github.com/docknimbus/nimbus/tree/master/.claude/skills/feature-workflow) |
| [gluo-developer](skills/gluo-developer) | Develop and maintain PHP REST APIs created with byjg/gluo: endpoints, models, persistence, DI configuration, migrations, authentication, OpenAPI, and tests. Use whenever working on a Gluo PHP application, even for generic requests such as "add a new endpoint" or "fix this bug" that do not name Gluo explicitly. Follow the project's installed dependency versions and existing conventions. Frontend-only work needs this skill only when it changes the API integration. | [byjg/php-gluo](https://github.com/byjg/php-gluo/tree/master/.claude/skills/gluo-developer) |

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

----
[Open source ByJG](https://opensource.byjg.com)
