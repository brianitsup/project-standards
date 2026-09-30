# project-standards

An [Agent Skill](https://agentskills.io) for Claude, Codex CLI, Gemini CLI, Cursor, opencode and other harnesses that read `SKILL.md`.

Install or sync agent instruction standards in a repository: PROJECT_WORKFLOW.md, AGENTS.md with a multi-agent delegation policy, and CLAUDE.md importing it.

## Note

This skill installs a personal set of agent instruction standards (`PROJECT_WORKFLOW.md`, `AGENTS.md`, `CLAUDE.md`). It expects the canonical `PROJECT_WORKFLOW.md` from its author's standards folder; adapt that path before using it for your own repositories.

## Install

With the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add brianitsup/project-standards
```

Manually (user scope for Codex, Gemini, Cursor, opencode):

```bash
git clone https://github.com/brianitsup/project-standards.git ~/my-skills/project-standards
ln -s ../../my-skills/project-standards ~/.agents/skills/project-standards
```

For Claude, upload the folder as a skill in Settings → Capabilities, or place it in `~/.claude/skills/project-standards` for Claude Code.

## License

MIT
