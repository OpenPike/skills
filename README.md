# OpenPike skills

Agent skills for [OpenPike](https://openpike.dev), the bring-your-own-key LLM gateway. A skill is a folder with a `SKILL.md` that a coding agent loads when your request matches it. These work in Claude Code, Codex and any agent that loads `SKILL.md` folders.

| Skill | What it does |
| --- | --- |
| [`openpike-setup`](skills/openpike-setup/SKILL.md) | Points a project or a coding agent at OpenPike: base URL, `OPENPIKE_API_KEY`, model names, tags, and what to do about OpenPike's `429` errors. Never asks for or writes a provider key. |
| `openpike-spend` | Planned: spend questions and budgets, once OpenPike has a management API. |

The commands and config in the skills are copied from the [OpenPike docs](https://openpike.dev/docs/).

## Install

Copy the skill folder into your agent's skills directory.

```bash
git clone https://github.com/OpenPike/skills.git
cd skills
```

**Claude Code**, for every project on this machine:

```bash
mkdir -p ~/.claude/skills
cp -r skills/openpike-setup ~/.claude/skills/
```

For one project, copy it to `.claude/skills/openpike-setup` in that project instead. Then ask: "Set this project up to use OpenPike." You can also run it with `/openpike-setup`.

**Codex**, for every project on this machine:

```bash
mkdir -p ~/.agents/skills
cp -r skills/openpike-setup ~/.agents/skills/
```

For one repo, copy it to `.agents/skills/openpike-setup` in that repo. Restart Codex if it was running.

On Windows, copy the folder with File Explorer or `Copy-Item -Recurse` to `$HOME\.claude\skills` or `$HOME\.agents\skills`.

## Before you use it

You need an OpenPike account, a provider key added on the **Providers** page of the dashboard, and a virtual key from the **Keys** page. The skill tells your agent to read the virtual key from the `OPENPIKE_API_KEY` environment variable. Provider keys are never typed into an agent.

## Contributing

Every command and config in a skill must match the OpenPike docs exactly, and the docs page it comes from should be linked. No em dashes. Open an issue or pull request.

## License

MIT, see [LICENSE](LICENSE). Security: see [SECURITY.md](SECURITY.md).
