# Agent Skills

[![Laravel Boost skill](https://badge.laravel.cloud/boost-badge.svg?style=flat)](https://badge.laravel.cloud/preview#?mode=boost)
[![License: MIT and Apache-2.0](https://img.shields.io/badge/license-MIT%20and%20Apache--2.0-blue.svg)](#license)

A focused collection of reusable skills for AI coding agents. The skills in this repository help agents review code changes, design Laravel application Actions and third-party boundaries, make Laravel AI SDK architecture decisions, and coordinate work in Solo-managed sessions.

## Available skills

| Skill | Purpose | Requirements |
| --- | --- | --- |
| [`pr-review-toolkit`](skills/pr-review-toolkit/SKILL.md) | Selects applicable pull-request review angles, delegates advisory review concurrently to independent specialists, and aggregates findings across correctness, tests, error handling, comments, type design, and simplification. | Git and an active delegation facility with independent agent contexts, either host-native or supplied by session orchestration; hosted pull-request lookup also needs a hosting CLI or API. |
| [`prvious-action-development`](skills/prvious-action-development/SKILL.md) | Creates, changes, reviews, and tests Laravel Actions as application use-case boundaries. | A Laravel application using the Prvious Laravel Action Pattern and [`prvious/result`](https://github.com/prvious/result). |
| [`prvious-integration-development`](skills/prvious-integration-development/SKILL.md) | Designs, implements, reviews, and tests boundaries around third-party APIs, SDKs, webhooks, and remote systems. | A Laravel application that integrates with a third-party or remote system. |
| [`prvious-ai-sdk-architecture`](skills/prvious-ai-sdk-architecture/SKILL.md) | Decides whether a Laravel AI SDK responsibility belongs in normal application code, a tool, an agent, a sub-agent, or an automation. | A Laravel application using the first-party [`laravel/ai`](https://github.com/laravel/ai) package. |
| [`using-solo`](skills/using-solo/SKILL.md) | Detects Solo-managed lead sessions and applies the appropriate orchestration workflow while leaving ordinary and worker sessions unchanged. | Solo MCP and at least one configured CLI Agent Tool for delegation. |

`prvious-action-development` owns application use-case architecture. The `prvious/result` package's `prvious-result-development` skill owns exact Result API behavior, generics, narrowing, composition, extraction, and Panic semantics. `prvious-integration-development` owns third-party boundaries, provider failure classification, and remote-operation decisions.

## Laravel Boost

Laravel applications using [Laravel Boost](https://github.com/laravel/boost) can add skills directly with Boost's [`boost:add-skill`](https://github.com/laravel/boost/blob/e4f651a973cb454616c36d324150b6a533913bd1/src/Console/AddSkillCommand.php#L35) Artisan command:

```sh
php artisan boost:add-skill prvious/skills
```

Boost discovers the skills in this repository and prompts you to choose which ones to install. You can also install all skills or select a specific skill non-interactively:

```sh
php artisan boost:add-skill prvious/skills --all
php artisan boost:add-skill prvious/skills --skill=SKILL_NAME
```

Replace `SKILL_NAME` with a name from the [available skills](#available-skills) table.

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
npx skills add prvious/skills --skill SKILL_NAME
```

Add `--global` to make a skill available across projects, or use `--agent` to target a specific supported agent:

```sh
npx skills add prvious/skills --skill SKILL_NAME --global --agent codex
```

## Usage

Once installed, a compatible agent discovers each skill from its name and description and loads it when a request matches.

Example requests for `pr-review-toolkit`:

```text
Review this pull request for high-confidence bugs and missing tests.

Audit the error handling and type invariants in my current diff.
```

Example requests for `prvious-action-development`:

```text
Move this controller's business logic into a typed Laravel Action.

Review this queued Action and make its expected errors and retryable exceptions explicit.
```

Example requests for `prvious-integration-development`:

```text
Review this Stripe integration and keep provider-specific behavior out of the application contract.

Decide whether this SDK call needs a concrete adapter or an application-owned gateway.
```

Example requests for `prvious-ai-sdk-architecture`:

```text
Review this Laravel AI feature and decide which responsibilities should be tools or agents.

Should this scheduled AI workflow be an automation, a sub-agent, or normal Laravel code?
```

`using-solo` is designed to run at the start of a conversation. It quietly continues without orchestration when Solo MCP is unavailable or the current process is not a Solo-managed lead.

## Contributing

Place each skill in its own directory under `skills/` with a `SKILL.md` containing valid YAML frontmatter and focused instructions. Keep supporting scripts, references, and agent configuration beside the skill that uses them.

## License

Except where a skill states otherwise, this project is open source under the [MIT License](LICENSE).

The adapted [`pr-review-toolkit`](skills/pr-review-toolkit/SKILL.md) is distributed under the Apache License 2.0; see its bundled [LICENSE](skills/pr-review-toolkit/LICENSE) and [NOTICE](skills/pr-review-toolkit/NOTICE).
