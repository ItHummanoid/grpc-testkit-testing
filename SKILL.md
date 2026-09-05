---
name: grpc-testkit-testing
description: Design, write, review, and troubleshoot Python pytest tests for gRPC services with the grpc-testkit library. Use for generated stubs or dynamic descriptors, synchronous or asynchronous clients, every RPC cardinality, status and metadata assertions, TLS/mTLS, explicit retries, snapshots, reflection, and in-process gRPC test servers. Do not activate for generic grpcio implementation work that does not use grpc-testkit.
license: MIT
compatibility: Compatible with Claude Code and OpenAI Codex on macOS, Linux, and Windows. Requires a Python project that uses grpc-testkit.
metadata:
  author: Yaroslav Chervatyuk
  source: https://gitlab.com/ItHummanoid/grpc-testkit
  verified-library-version: "0.3.1"
---

# grpc-testkit Testing

Build repository-grounded Python gRPC tests around `grpc-testkit`. Preserve the project's existing pytest architecture, generated protobuf workflow, configuration model, and reporting conventions.

## Operating sequence

1. Inspect the repository before proposing code: dependency manifests, pytest configuration, fixtures, existing gRPC clients/tests, generated `*_pb2.py` and `*_pb2_grpc.py`, `.proto` sources, and environment configuration.
2. Detect the installed or pinned `grpc-testkit` version. This skill is verified against `0.3.1`; if the project uses another version, inspect that version's public API before generating code.
3. Trace each target RPC end to end: service and method, cardinality, request/response types, metadata, authentication, deadlines, retry safety, expected statuses, and cleanup or isolation needs.
4. Choose the least dynamic schema path that fits the repository: generated stubs first, then descriptors, server reflection, or local proto compilation. Never regenerate protobuf files unless the user authorizes it and the project workflow is known.
5. Match the application's concurrency model: use `Session`/`ServiceClient` for synchronous suites and `AsyncSession`/`AsyncServiceClient` for async suites. Do not mix blocking clients into an async test merely for convenience.
6. Reuse one session and a typed service-object wrapper through pytest fixtures. Keep request construction, transport calls, and assertions in distinct layers when the existing suite does so.
7. Cover the contract, not only the happy path: valid responses, validation errors, authentication/authorization, not-found/conflict cases, deadlines, metadata, and relevant streaming termination behavior.
8. Assert gRPC semantics deeply: status code, protobuf fields, details, metadata, and business invariants. Avoid assertions that only check truthiness or message existence.
9. Verify locally without network effects first. Run live integration tests only after the user confirms a non-production target, credentials, data impact, and scope.

## Non-negotiable rules

- Prefer generated stubs for maintained typed suites. Use reflection or proto compilation for discovery, black-box testing, or repositories without generated code.
- Keep strict error handling enabled by default. Catch `GrpcCallError` when the response must be inspected, or use `raise_on_error=False` only for intentional status matrices.
- Enable retries only with an explicit `RetryPolicy` and `idempotent=True`. Treat retries as valid only for unary-unary operations unless the installed library version documents otherwise.
- Consume or close streaming calls before reading their terminal response. Bound stream collection by a count, deadline, cancellation condition, or domain terminator.
- Accept that early client cancellation may finish as `CANCELLED` or `OK` because of transport timing; assert the contract the service actually guarantees.
- Load endpoints, tokens, certificates, and metadata from the repository's configuration layer. Never hardcode or print secrets, and never target production without explicit authorization.
- Preserve public protobuf types at service boundaries. Use `response_type=` when dynamic invocation needs a type witness or runtime response validation.
- Do not install packages, enable dormant gRPC infrastructure, regenerate protobuf code, or change CI configuration without explicit scope and impact review.

## Reference routing

Read only the references needed for the task:

- Public objects, extras, schema-loading choices, and version drift: [references/library-api.md](references/library-api.md)
- Repository structure, service objects, fixtures, and test design: [references/test-architecture.md](references/test-architecture.md)
- Async calls and all four streaming cardinalities: [references/async-and-streaming.md](references/async-and-streaming.md)
- Profiles, TLS/mTLS, auth, metadata, retries, deadlines, and safe targets: [references/configuration-and-security.md](references/configuration-and-security.md)
- Evidence-based verification and failure triage: [references/verification.md](references/verification.md)

## Integration boundaries

This standalone skill is library-specific. It may consume repository findings from an audit or source-inventory skill and may complement an architecture skill, but it must not depend on another skill or plugin. When a repository already uses `grpc-testkit`, extend its wrappers and fixtures instead of scaffolding a competing generic gRPC client layer.

## Expected output

Provide:

1. repository evidence and assumptions;
2. the chosen schema/client strategy and why it fits;
3. affected files and dependency/configuration impact;
4. test cases, isolation, and cleanup behavior;
5. implementation or a copy-paste-ready proposal, according to the user's authorization;
6. exact verification commands, distinguishing executed checks from recommended live checks.

## Source and license

This independently authored skill is MIT-licensed. It teaches the separately distributed [grpc-testkit](https://gitlab.com/ItHummanoid/grpc-testkit) library. The verified `0.3.1` release remains Apache-2.0 licensed; the library source switches to MIT beginning with its next release. Do not copy substantial library source or documentation into generated output; preserve the license applicable to the redistributed version.
