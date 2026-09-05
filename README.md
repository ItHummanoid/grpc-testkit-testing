# grpc-testkit-testing skill

An independent agent skill for designing, writing, reviewing, and troubleshooting Python gRPC tests with [grpc-testkit](https://gitlab.com/ItHummanoid/grpc-testkit).

The skill is MIT-licensed. The verified library release `0.3.1` is a separate
Apache-2.0 project; its next release switches to MIT and keeps its own license file.

## Install as a standalone skill

Publish this directory as the repository root, or place it under `skills/grpc-testkit-testing/` in a multi-skill repository.

From GitHub:

```bash
npx skills add https://github.com/<owner>/<repository> --skill grpc-testkit-testing
```

From GitLab or another git host supported by the installer:

```bash
npx skills add https://gitlab.com/<owner>/<repository> --skill grpc-testkit-testing
```

The installer supports GitLab sources, but the public `skills.sh` catalog currently presents skills through GitHub repository pages. Maintain a GitHub mirror if discoverability on `skills.sh` is required; GitLab can remain the canonical home of `grpc-testkit` itself.

## Add to python-qa-agent

Copy this entire directory to:

```text
python-qa-agent/skills/grpc-testkit-testing/
```

The skill has no dependency on other plugin skills, so the same files can be published standalone. During a future plugin integration, also update the plugin's routing metadata and inventory tests so gRPC requests that mention `grpc-testkit` select this skill instead of a generic gRPC scaffold.

Recommended integration work in `python-qa-agent`:

1. add `grpc-testkit`, `grpc testkit`, and closely related intent signals to both selector paths;
2. route existing `grpc-testkit` repositories to this skill while retaining the generic architecture skill for other clients;
3. add inbound references from the central orchestrator and related gRPC skill;
4. update skill-count, reachability, and metadata tests;
5. preserve this `LICENSE` and source attribution.

No plugin files are modified by this standalone package.

## Contents

- `SKILL.md` — activation criteria and operating workflow;
- `references/library-api.md` — verified public API and compatibility baseline;
- `references/test-architecture.md` — service-object and pytest architecture;
- `references/async-and-streaming.md` — async and streaming rules;
- `references/configuration-and-security.md` — credentials, TLS, retries, and deadlines;
- `references/verification.md` — safe verification sequence;
- `agents/openai.yaml` — optional Codex UI metadata;
- `LICENSE` — MIT license for this skill.
