# Claude Skills

A personal collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills, organized by category and installable into any project.

## Quick Start

1. Clone this repo:
   ```bash
   git clone https://github.com/<your-username>/claude.git
   cd claude
   ```
2. Open Claude Code in this directory:
   ```bash
   claude
   ```
3. Ask Claude to install a skill:
   > *"Install the dotnet/project-creator skill into /path/to/my-project"*
   >
   > *"Install the dotnet/project-creator skill globally"*

The `skill-installer` skill handles the rest — it copies the skill files and registers them in the appropriate `CLAUDE.md`.

## Available Skills

### dotnet

| Skill | Description |
|---|---|
| [dotnet/project-creator](skills/dotnet/project-creator/SKILL.md) | Scaffold a .NET Core solution + project using the `dotnet` CLI. Supports C#, F#, and VB. |

## Installation Scopes

| Scope | Where skills are copied | Available in |
|---|---|---|
| **Local** | `<project>/.claude/skills/` | That project only |
| **Global** | `~/.claude/skills/` | All Claude Code sessions |

## Repo Structure

```
.claude/
  skills/
    skill-installer/   # Meta-skill: installs other skills from this repo
skills/
  dotnet/
    project-creator/   # Scaffold .NET solutions
```

