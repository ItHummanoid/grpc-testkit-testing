# Configuration, security, retries, and deadlines

## Configuration discovery

Reuse the repository's settings and secret-loading path. Determine:

- target syntax and service authority;
- secure or insecure channel policy;
- root CA and client certificate sources;
- bearer token or static metadata provider;
- default and per-call deadlines;
- retry policy and idempotency classification;
- proxy, DNS, and load-balancer behavior;
- allowed non-production environments.

Do not infer that a target is safe from its hostname. Require explicit confirmation before a live test mutates remote state.

## Profiles and secrets

Profiles are suitable for non-secret defaults and named environments. Resolve secrets at runtime through the existing environment, vault, or CI mechanism. Keep tokens, cookies, private keys, and authorization metadata out of:

- source control;
- snapshots;
- pytest parameter IDs;
- Allure or other report attachments;
- exception messages and debug logs.

Use the library's redaction support where diagnostics may include metadata.

## TLS and mTLS

For TLS tests, validate certificate and authority handling rather than disabling verification. For mTLS, load client certificates from an approved secret path and test failure behavior using disposable credentials or an in-process server. Never embed PEM content in a test or skill output.

## Authentication and metadata

Use `BearerToken`, `AsyncBearerToken`, or `StaticMetadata` when they match the repository's authentication model. Keep cross-cutting metadata in session/profile configuration or interceptors; keep scenario-specific headers on the individual call.

Test both presence and server interpretation of required metadata. Do not attach the raw authorization value to reports.

## Retries

Retries can duplicate side effects. Require all of the following:

1. a concrete `RetryPolicy`;
2. a reviewed set of retryable status codes;
3. explicit `idempotent=True` at the call boundary;
4. unary-unary cardinality for the verified library version;
5. an overall deadline that bounds all attempts.

Test attempt count and final diagnostics with an in-process server. Do not classify create/update RPCs as idempotent from their names alone.

## Deadlines

Every potentially blocking test needs a bounded deadline. Use `DeadlineBudget` when multiple calls or stream phases must share one end-to-end budget. Avoid fixed sleeps; produce deterministic delays on the test server or use bounded polling for eventual consistency.

## Target and pytest options

Use `Target` when the library's parsed target model improves validation. If the pytest plugin is enabled, inspect its registered command-line options and fixtures in the installed version before using them. Do not shadow an existing repository option with a new `--grpc-*` option.

## Schema acquisition

- Generated modules: follow the repository's existing generation command and pinned toolchain.
- Reflection: confirm it is enabled and allowed in the target environment.
- Local proto: require the `proto` extra and known import roots.
- Contract checks: use `BufContractChecker` only when Buf is already available or its installation has been approved.

Treat protobuf generation and compatibility gates as build-system changes, not incidental test edits.

## Reporting

Attach sanitized request/response evidence only when it helps diagnosis. Prefer protobuf JSON/text with secret and volatile fields redacted. Record status, details, elapsed time, attempts, and safe metadata; omit credentials and certificate material.
