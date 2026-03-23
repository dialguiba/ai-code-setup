# ai-code-setup

Personal AI coding skills and configurations for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Skills

| Skill | Description |
|---|---|
| [plan-commits](skills/plan-commits/) | Analyze uncommitted changes and organize them into logical, granular Conventional Commits |
| [grill-me](skills/grill-me/) | Interview the user relentlessly about a plan or design until reaching shared understanding |
| [write-a-prd](skills/write-a-prd/) | Create a PRD through user interview, codebase exploration, and module design |
| [prd-to-issues](skills/prd-to-issues/) | Break a PRD into independently-grabbable vertical slices |
| [tdd](skills/tdd/) | Test-driven development with red-green-refactor loop |

## Commands

| Command | Description |
|---|---|
| [plannotator-annotate](commands/plannotator-annotate.md) | Open interactive annotation UI for a markdown file |
| [plannotator-last](commands/plannotator-last.md) | Annotate the last rendered assistant message |
| [plannotator-review](commands/plannotator-review.md) | Open interactive code review for current changes or a PR URL |

## Installation

Copy skills and commands into your Claude Code directory:

```bash
# Skills
cp -r skills/* ~/.claude/skills/

# Commands
cp commands/* ~/.claude/commands/
```

## Usage

In Claude Code, use slash commands:

```
/plan-commits
/grill-me
/write-a-prd
/prd-to-issues
/tdd
/plannotator-review
/plannotator-annotate
/plannotator-last
```
