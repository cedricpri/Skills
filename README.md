# cedric-skills

Personal Claude Code skills, packaged as a plugin.

## Install (once per machine)

```bash
# Global coding standards, loaded in every session
ln -s "$(pwd)/CLAUDE.global.md" ~/.claude/CLAUDE.md

# In Claude Code
/plugin marketplace add <github-user>/cedric-skills
/plugin install cedric@cedric
```

## Update after pushing changes

```
/plugin marketplace update cedric
```

## Skills

| Skill | Invoked by | Purpose |
|---|---|---|
| `/cedric:setup-repo` | you | Detect stack, add check and audit commands, tool config |
| `/cedric:plan` | you | Interview, then write a spec with vertical-slice tasks |
| `/cedric:implement` | you | Implement a spec test-first, checking after each task |
| `/cedric:review` | you | Review a diff on correctness and standards |
| `/cedric:cleanup` | you | Verified dead-code removal and simplification |
| `/cedric:security-audit` | you | Scans plus stack-specific manual security review |
| `/cedric:debug` | you or Claude | Reproduce, minimize, verify, fix, regression test |
| `/cedric:ml-review` | you or Claude | Leakage, silent data bugs, evaluation, reproducibility |

## Subagent

`cedric:implementer` runs on Sonnet and writes the code. The main conversation (Opus) plans, writes the brief, delegates, then verifies the diff and runs the checks. `plan`, `review`, `debug` and `ml-review` stay on the main model because they are reasoning work.
