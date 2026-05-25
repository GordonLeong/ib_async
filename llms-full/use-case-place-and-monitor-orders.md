# Use Case: Place and Monitor Orders

## What You're Trying to Do

You want to submit an order and track status/fills from submission through completion.

## Complete Working Example

```python
from ib_async import IB, Stock, MarketOrder

ib = IB()
ib.connect("127.0.0.1", 7497, clientId=8)

try:
    contract = Stock("AAPL", "SMART", "USD")
    ib.qualifyContracts(contract)

    order = MarketOrder("BUY", 1)
    trade = ib.placeOrder(contract, order)

    def on_status(updated_trade):
        status = updated_trade.orderStatus.status
        print("status:", status, "filled:", updated_trade.orderStatus.filled)

    ib.orderStatusEvent += on_status

    while trade.isActive():
        ib.sleep(0.2)

    print("done", trade.orderStatus.status)
    for fill in trade.fills:
        print("fill", fill.execution.shares, fill.execution.price)
finally:
    ib.disconnect()
```

## Step-by-Step Breakdown

1. Connect and qualify the contract first.
2. Build `MarketOrder` with side and quantity.
3. `placeOrder` returns a `Trade` object that will mutate as updates arrive.
4. Register `orderStatusEvent` callback to observe transitions.
5. Poll `trade.isActive()` with `ib.sleep` to keep processing events.
6. After completion, inspect terminal status and fill list.

## Variations

- Pre-flight margin check: `await ib.whatIfOrderAsync(contract, order)`.
- Bracket orders: use `ib.bracketOrder(...)` then place each returned child order.
- Async status waiting: await explicit events/futures in your app instead of polling loop.

## Gotchas Specific to This Use Case

- Disconnecting immediately after submit can drop outbound packets; add a short `ib.sleep(1)` before disconnect for short-lived sessions.
- Event handlers should not recursively fire many new requests.
- Duplicate `clientId` across sessions causes connect/order conflicts.
