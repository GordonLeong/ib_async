# Capabilities Deep Dive

## Runtime-Switchable Error Strictness
**What it enables**: Turn silent empty results into explicit failures.
**Why it matters**: Production automation needs deterministic error handling.

```python
import ib_async as ibi

ib = ibi.IB()
ib.RaiseRequestErrors = True
# failing requests now raise ibi.RequestError
```

**Key details**:
- Verified in tests with `whatIfOrderAsync` on bad contracts.
- Error objects include IBKR code fields for branching logic.

## Event-Driven Ticker Batching
**What it enables**: Process all updated tickers per network packet.
**Why it matters**: Efficient UI/stream processing without per-field callbacks.

```python
def on_pending(tickers):
    for t in tickers:
        print(t.contract.symbol, t.last)

ib.pendingTickersEvent += on_pending
```

**Key details**:
- Tickers are mutable objects updated in place.
- Batch event fires after packet processing.

## Built-In Request Throttling
**What it enables**: Avoid request bursts exceeding IB pacing limits.
**Why it matters**: Reduces pacing errors without external rate-limit code.

```python
from ib_async.client import Client
print(Client.MaxRequests, Client.RequestsInterval)  # defaults: 45 / 1s
```

**Key details**:
- Throttle behavior is part of `Client` internals.
- Can be tuned via class attributes if needed.
