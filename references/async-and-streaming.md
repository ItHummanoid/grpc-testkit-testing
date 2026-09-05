# Async and streaming behavior

## Match the concurrency boundary

Use `AsyncSession` and `AsyncServiceClient` in async applications and pytest suites. Await unary calls and close the session using its async context manager. Do not wrap the synchronous API in an executor unless the repository already has a justified compatibility boundary.

```python
import grpc
import pytest
from grpc_testkit import AsyncServiceClient, AsyncSession, assert_fields, assert_status

from app.proto import orders_pb2, orders_pb2_grpc


@pytest.mark.asyncio
async def test_get_order(async_grpc_target, order_request):
    async with AsyncSession(async_grpc_target) as session:
        service = AsyncServiceClient(session, orders_pb2_grpc.OrdersStub)
        response = await service.unary(
            "GetOrder",
            order_request,
            response_type=orders_pb2.Order,
        )

    assert_status(response, grpc.StatusCode.OK)
    assert response.message is not None
    assert_fields(response.message, order_id=order_request.order_id)
```

Confirm whether the repository uses `pytest-asyncio`, another async plugin, or native async test support before choosing markers and fixtures.

## Cardinality map

| RPC shape | Input | Output | Test focus |
| --- | --- | --- | --- |
| unary-unary | one request | one response | status, fields, metadata, deadlines, explicit safe retries |
| unary-stream | one request | response stream | ordering, count, termination, mid-stream failure |
| stream-unary | request iterator/async iterator | one response | backpressure, aggregation, partial input failure |
| stream-stream | request iterator/async iterator | response stream | interleaving, cancellation, half-close, terminal status |

## Terminal response contract

For streaming calls, the final `Response` cannot be trusted until the stream is fully consumed or explicitly closed. Tests must:

1. bound consumption by an expected count, domain terminator, or deadline;
2. collect only what the assertion needs;
3. close or cancel in `finally` when exiting early;
4. inspect final status and trailing metadata only after terminal state.

Never convert an unbounded stream to `list(...)` without an independent termination guarantee.

## Streaming failure cases

Exercise controlled cases where relevant:

- error before the first item;
- error after one or more items;
- deadline before and during iteration;
- invalid item in a client stream;
- server closes early;
- client half-closes;
- client cancellation;
- bidi responses arriving at a different cadence from requests.

Preserve both received messages and terminal diagnostics when the API exposes them.

## Cancellation race

An early client close can race with normal server completion. A terminal code of `CANCELLED` or `OK` may both be valid at the transport level. Do not weaken assertions blindly: first identify whether the product contract guarantees cancellation, normal completion, or only prompt resource release.

## Request iterators

Keep generators deterministic and side-effect-light. For async streams, handle cancellation in the producer and avoid hidden sleeps. Coordinate timing with events, controlled server behavior, or bounded polling rather than arbitrary delays.
