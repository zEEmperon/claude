# Claude Skills Repository

This is a personal collection of Claude Code skills.

## Structure

```
.claude/skills/        # Meta-skills: auto-loaded when this repo is open in Claude Code
  skill-installer/     # Install a skill from this repo into a project or globally
  skill-remover/       # Remove an installed skill
  skill-creator/       # Scaffold a new skill in this repo
  skill-runner/        # Run a skill from /skills without installing it
  skill-upgrader/      # Update an installed skill to the latest catalog version
skills/<category>/     # Installable skill catalog
  <skill-name>/
    SKILL.md
```

## Available Skills

| Path | Name | Description |
|---|---|---|
| `skills/dotnet/project-creator` | dotnet-project-creator | Scaffold a .NET solution + project using the dotnet CLI |

