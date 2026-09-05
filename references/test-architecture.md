# Test architecture

## Discovery checklist

Before writing tests, locate:

- dependency and lock files;
- pytest options, markers, plugins, and async mode;
- existing channel/session fixtures and teardown behavior;
- generated protobuf modules and their generation source;
- service-object or API-client conventions;
- endpoint, TLS, token, and metadata configuration;
- existing test-data factories and cleanup mechanisms;
- CI selection, reporting, and non-production safeguards.

Trace the specific RPC declaration and server behavior. Names alone are insufficient to infer idempotency, authorization, eventual consistency, or stream termination.

## Recommended structure

Fit the repository first. When no convention exists, a small structure is enough:

```text
tests/
  grpc/
    clients/
      orders_client.py
    conftest.py
    test_orders.py
```

Keep generated protobuf code outside hand-written test modules and never edit it directly.

## Typed service object

Adapt imports and method names to the installed library version and generated code:

```python
from grpc_testkit import Response, ServiceClient, Session

from app.proto import orders_pb2, orders_pb2_grpc


class OrdersClient:
    def __init__(self, session: Session) -> None:
        self._service = ServiceClient(session, orders_pb2_grpc.OrdersStub)

    def get_order(
        self,
        order_id: str,
        *,
        correlation_id: str,
    ) -> Response[orders_pb2.Order]:
        request = orders_pb2.GetOrderRequest(order_id=order_id)
        return self._service.unary(
            "GetOrder",
            request,
            response_type=orders_pb2.Order,
            metadata=(("x-correlation-id", correlation_id),),
        )
```

The wrapper owns protocol details. Tests own scenarios and business assertions. If the repository already has a client wrapper, add the RPC there instead of creating another abstraction.

## Fixture lifecycle

Prefer a session- or module-scoped transport fixture with deterministic teardown, plus narrower service fixtures:

```python
import pytest
from grpc_testkit import Session


@pytest.fixture(scope="session")
def grpc_session(grpc_target: str):
    with Session(grpc_target) as session:
        yield session


@pytest.fixture
def orders_client(grpc_session: Session) -> OrdersClient:
    return OrdersClient(grpc_session)
```

Reuse the repository's established configuration fixture for `grpc_target`; do not introduce a second environment-variable convention without a reason.

## Scenario design

For each RPC, consider:

- minimally valid request and full business-valid request;
- omitted, malformed, boundary, and unknown identifiers;
- unauthenticated and unauthorized calls where safe;
- idempotency or duplicate submission behavior;
- deadline exceeded and unavailable behavior where controlled;
- initial/trailing metadata and correlation IDs;
- ordering, pagination, and repeated fields;
- server-stream completion, client-stream aggregation, and bidi interaction;
- state isolation and cleanup for data-changing operations.

Use unique test data. Prefer public cleanup APIs or disposable in-process services; never add destructive database cleanup merely to simplify a test.

## Assertion depth

A useful test normally proves more than `StatusCode.OK`:

```python
import grpc
from grpc_testkit import assert_fields, assert_metadata_contains, assert_status


correlation_id = f"test-{order_id}"
response = orders_client.get_order(order_id, correlation_id=correlation_id)

assert_status(response, grpc.StatusCode.OK)
assert response.message is not None
assert_fields(response.message, order_id=order_id, customer__id=customer_id)
assert_metadata_contains(response, x_correlation_id=correlation_id)
```

Add domain invariants such as totals, state transitions, timestamps, and cross-call consistency. Avoid brittle equality on server-generated or unordered fields.

## Payloads and snapshots

Use `PayloadFactory` when descriptor-aware generation reduces duplication, but keep explicit values for fields central to the scenario. Seed randomness when reproducibility matters.

Use snapshots for stable protobuf structures, not volatile IDs, timestamps, tokens, or unordered collections. Normalize or exclude volatile fields using supported library hooks rather than post-processing opaque serialized text.

## In-process servers

Prefer the library's in-process server utilities for:

- client wrapper unit tests;
- deterministic status, metadata, retry, and deadline cases;
- all four RPC cardinalities;
- protocol failures that should not be injected into a shared environment.

Keep at least a narrow live integration check when generated code, routing, credentials, proxies, or deployed interceptors are part of the risk.
