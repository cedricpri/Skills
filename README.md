# cedric-skills

Personal Claude Code skills, packaged as a plugin.

## Install (once per machine)

```bash
# Global coding standards, loaded in every session
ln -s "$(pwd)/CLAUDE.global.md" ~/.claude/CLAUDE.md

# In Claude Code
/plugin marketplace add cedricpri/Skills
/plugin install cedric@cedric
```

## Update after pushing changes

```
/plugin marketplace update cedric
```

## Skills

| Skill | Invoked by | Purpose |
|---|---|---|
| `/cedric:plan` | you | Interview, then write a spec with vertical-slice tasks |
| `/cedric:implement` | you | Implement a spec test-first, verifying after each task |
| `/cedric:debug` | you or Claude | Root-cause debugging: errors, production-only bugs, UI, data, performance |
| `/cedric:review` | you or Claude | In-depth review of a diff, branch, PR, folder or the whole codebase |
| `/cedric:cleanup` | you or Claude | Verified dead-code removal, feature removal and simplification |
| `/cedric:security-audit` | you or Claude | Scans, attack surface and manual review of code and self-hosted servers |

Stack-specific checklists live in each skill's `references/` folder and are loaded only when needed.
