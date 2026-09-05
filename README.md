# grpc-testkit-testing skill

An independent agent skill for designing, writing, reviewing, and troubleshooting Python gRPC tests with [grpc-testkit](https://gitlab.com/ItHummanoid/grpc-testkit).

The skill is MIT-licensed. The verified library release `0.3.1` is a separate
Apache-2.0 project; its next release switches to MIT and keeps its own license file.

## Supported agents and platforms

The skill follows the open Agent Skills format and supports:

- OpenAI Codex;
- Claude Code;
- macOS, Linux, and Windows.

It contains no platform-specific scripts. The installation commands below are single-line commands that work in Bash, zsh, PowerShell, and Command Prompt. They require Node.js with `npx` available.

## Install for OpenAI Codex

From GitHub:

```bash
npx skills add ItHummanoid/grpc-testkit-testing --skill grpc-testkit-testing --agent codex --global
```

From GitLab:

```bash
npx skills add https://gitlab.com/ItHummanoid/grpc-testkit-testing.git --skill grpc-testkit-testing --agent codex --global
```

## Install for Claude Code

From GitHub:

```bash
npx skills add ItHummanoid/grpc-testkit-testing --skill grpc-testkit-testing --agent claude-code --global
```

From GitLab:

```bash
npx skills add https://gitlab.com/ItHummanoid/grpc-testkit-testing.git --skill grpc-testkit-testing --agent claude-code --global
```

If Claude Code was already running and `~/.claude/skills/` did not previously exist, restart Claude Code after installation. The skill can then be invoked explicitly with:

```text
/grpc-testkit-testing
```

Verify the installations with:

```bash
npx skills list --global --agent codex
npx skills list --global --agent claude-code
```

The installer supports GitLab sources, but the public `skills.sh` catalog currently presents skills through GitHub repository pages. Maintain a GitHub mirror if discoverability on `skills.sh` is required; GitLab can remain the canonical home of `grpc-testkit` itself.

## Contents

- `SKILL.md` — activation criteria and operating workflow;
- `references/library-api.md` — verified public API and compatibility baseline;
- `references/test-architecture.md` — service-object and pytest architecture;
- `references/async-and-streaming.md` — async and streaming rules;
- `references/configuration-and-security.md` — credentials, TLS, retries, and deadlines;
- `references/verification.md` — safe verification sequence;
- `agents/openai.yaml` — optional OpenAI Codex UI metadata, ignored by other compatible agents;
- `LICENSE` — MIT license for this skill.
