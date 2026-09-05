# Library API and compatibility

## Verified baseline

This reference was checked against:

- source: `https://gitlab.com/ItHummanoid/grpc-testkit`
- release: `0.3.1`
- commit: `bc2a4b184c890b7a9f92dd097ba12f43e3af9be7`
- Python: `>=3.10,<3.15`
- core dependencies: `grpcio>=1.66,<2` and `protobuf>=5.27,<7`

The verified library release `0.3.1` is Apache-2.0 licensed. Its source switches to MIT
beginning with the next release; the surrounding skill is a separate MIT-licensed work.

## Installation extras

Choose the smallest extra that supports the requested behavior, and preserve the project's dependency pinning style.

| Extra | Use |
| --- | --- |
| `proto` | Load and compile local `.proto` definitions |
| `reflection` | Discover descriptors from a reflection-enabled server |
| `health` | Exercise the standard gRPC health service |
| `config` | Load richer configuration/profile files |
| `pytest` | Pytest plugin, options, and fixtures |
| `allure` | Allure integration |
| `status` | Decode rich `google.rpc.Status` details |
| `otel` | OpenTelemetry integration |
| `security` | Security-related helpers |
| `full` | All optional integrations; use only when the suite needs them |

Do not add an extra speculatively. Confirm its transitive dependencies, Python compatibility, CI installation source, and license policy.

## Client and schema choices

Use one of these paths:

1. **Generated stubs** — preferred for long-lived, typed suites. Pass the generated stub class to `ServiceClient(session, StubType)` or `AsyncServiceClient(session, StubType)`.
2. **Descriptors** — useful when descriptors already exist but generated stubs do not.
3. **Server reflection** — useful for black-box discovery when reflection is enabled and approved.
4. **Local proto loading** — useful for exploratory tests or repositories whose official workflow compiles schemas at runtime.

The principal public entry points are:

- `Session`, `AsyncSession`
- `ServiceClient`, `AsyncServiceClient`
- `request`, `arequest`
- `connect`, `aconnect`

Service helpers cover all cardinalities:

- `unary(...)`
- `server_stream(...)`
- `client_stream(...)`
- `bidi(...)`

Callable-first variants use `invoke*`. Dynamic calls may require `response_type=` for response construction or runtime type validation.

## Response and assertions

Treat `Response` as the stable observation object. Depending on call type and completion state, it exposes the protobuf message, gRPC code, details, initial and trailing metadata, elapsed time, attempts, and related diagnostics.

Prefer the library's standalone assertion functions when they express the contract clearly:

- `assert_status(...)`
- `assert_fields(...)`, including nested paths such as `customer__id`
- `assert_message_equal(...)`
- `assert_metadata_contains(...)`
- `expect_status(...)`

Strict mode raises `GrpcCallError` for non-OK calls and retains the `Response`. For a negative matrix, `raise_on_error=False` can make status assertions linear and explicit.

## Additional capabilities

Use these only when the scenario calls for them:

- `PayloadFactory` for protobuf-aware request data;
- snapshots for stable, reviewable protobuf output;
- in-process servers for isolated transport-level tests;
- reflection and health clients for infrastructure checks;
- `DeadlineBudget` for a shared time budget across calls;
- rich-status decoding for structured service errors;
- `BufContractChecker` for schema compatibility;
- `Target` for parsed and validated endpoint configuration;
- redaction helpers for safe diagnostics.

## Handling version drift

If the repository does not use `0.3.1`:

1. inspect the installed distribution metadata or the exact tagged source;
2. confirm constructor and method signatures used by the proposed test;
3. inspect that version's changelog and public exports;
4. avoid silently using APIs present only in this reference;
5. state the verified version in the delivery.
