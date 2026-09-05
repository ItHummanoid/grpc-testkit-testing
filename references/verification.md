# Verification workflow

Run the narrowest safe checks that match the change and the repository's configured tooling.

## 1. Static and import checks

Use existing project commands where available. Otherwise, suitable narrow checks include:

```bash
python -m compileall path/to/changed/tests path/to/changed/clients
python -c "import path.to.changed_module"
```

Beware of import-time settings or network initialization. Do not claim an import check if it was skipped for safety.

## 2. Collection

Confirm that pytest discovers the intended node without invoking a remote service:

```bash
pytest --collect-only -q path/to/test_module.py
```

Collection can still execute import-time code; inspect configuration first.

## 3. Isolated transport tests

Run in-process server tests before live tests:

```bash
pytest -q path/to/test_client_unit.py
```

Cover status propagation, metadata, deadlines, retry attempts, serialization, cardinality, stream cleanup, and async teardown.

## 4. Narrow live integration node

Only after explicit confirmation of a sanitized non-production environment, credentials, data impact, and cleanup:

```bash
pytest -q path/to/test_grpc.py::TestService::test_rpc_case
```

Then run the smallest relevant marker or directory suite already used by the repository. Do not invent marker names without registering them.

## 5. Optional project gates

Run formatting, lint, typing, coverage, protobuf generation, Buf checks, or Allure generation only if the repository configures them or the user approves their introduction. Report every command actually run and its result.

## Failure triage

Classify failures before changing tests:

- schema mismatch or stale generated code;
- wrong target/authority or TLS configuration;
- authentication/metadata failure;
- gRPC status or rich-status contract mismatch;
- deadline or retry-budget exhaustion;
- stream not consumed or closed;
- server-side business failure;
- test-data collision or missing cleanup;
- version drift in `grpc-testkit`.

Preserve `GrpcCallError.response` diagnostics and sanitize them before sharing.

## Completion report

State:

- library version and schema path used;
- files changed;
- tests added or updated;
- checks run with exact outcomes;
- live checks intentionally not run;
- remaining environment, data, security, or compatibility risks.
