# Agent Skills

[![Laravel Boost skill](https://badge.laravel.cloud/boost-badge.svg?style=flat)](https://badge.laravel.cloud/preview#?mode=boost)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

A focused collection of reusable skills for AI coding agents. The skills in this repository help agents make better Laravel AI architecture decisions and coordinate work in Solo-managed sessions.

## Available skills

| Skill | Purpose | Requirements |
| --- | --- | --- |
| [`laravel-ai-architecture`](skills/laravel-ai-architecture/SKILL.md) | Decides whether a Laravel AI responsibility belongs in normal application code, a tool, an agent, a sub-agent, or an automation. | A Laravel application using the first-party [`laravel/ai`](https://github.com/laravel/ai) package. |
| [`using-solo`](skills/using-solo/SKILL.md) | Detects Solo-managed lead sessions and applies the appropriate orchestration workflow while leaving ordinary and worker sessions unchanged. | Solo MCP and at least one configured CLI Agent Tool for delegation. |

## Laravel Boost

Laravel applications using [Laravel Boost](https://github.com/laravel/boost) can add skills directly with Boost's [`boost:add-skill`](https://github.com/laravel/boost/blob/e4f651a973cb454616c36d324150b6a533913bd1/src/Console/AddSkillCommand.php#L35) Artisan command:

```sh
php artisan boost:add-skill prvious/skills
```

Boost discovers the skills in this repository and prompts you to choose which ones to install. You can also install all skills or select a specific skill non-interactively:

```sh
php artisan boost:add-skill prvious/skills --all
php artisan boost:add-skill prvious/skills --skill=laravel-ai-architecture
```

Use `--list` to see the available skills without installing them:

```sh
php artisan boost:add-skill prvious/skills --list
```

## Skills CLI

Install the collection with the [Skills CLI](https://github.com/vercel-labs/skills):

```sh
npx skills add prvious/skills
```

To install only one skill:

```sh
npx skills add prvious/skills --skill laravel-ai-architecture
npx skills add prvious/skills --skill using-solo
```

Add `--global` to make a skill available across projects, or use `--agent` to target a specific supported agent:

```sh
npx skills add prvious/skills --skill laravel-ai-architecture --global --agent codex
```

## Usage

Once installed, a compatible agent discovers each skill from its name and description and loads it when a request matches.

Example requests for `laravel-ai-architecture`:

```text
Review this Laravel AI feature and decide which responsibilities should be tools or agents.

Should this scheduled AI workflow be an automation, a sub-agent, or normal Laravel code?
```

`using-solo` is designed to run at the start of a conversation. It quietly continues without orchestration when Solo MCP is unavailable or the current process is not a Solo-managed lead.

## Contributing

Place each skill in its own directory under `skills/` with a `SKILL.md` containing valid YAML frontmatter and focused instructions. Keep supporting scripts, references, and agent configuration beside the skill that uses them.

## License

This project is open source under the [MIT License](LICENSE).
