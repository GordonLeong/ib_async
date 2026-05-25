# Use Case: Run an asyncio-Native Workflow

## What You're Trying to Do

You already use asyncio and need IB requests without blocking your event loop.

## Complete Working Example

```python
import asyncio
from ib_async import IB, Forex


async def main():
    ib = IB()
    await ib.connectAsync("127.0.0.1", 7497, clientId=9)
    try:
        contract = Forex("EURUSD")
        await ib.qualifyContractsAsync(contract)

        bars = await ib.reqHistoricalDataAsync(
            contract,
            endDateTime="",
            durationStr="1 D",
            barSizeSetting="1 hour",
            whatToShow="MIDPOINT",
            useRTH=True,
        )
        print("bars:", len(bars))

        summary = await ib.accountSummaryAsync()
        print("account fields:", len(summary))
    finally:
        ib.disconnect()


if __name__ == "__main__":
    asyncio.run(main())
```

## Step-by-Step Breakdown

1. Create one `IB` instance inside your async runtime.
2. `connectAsync` performs socket setup and readiness handshake.
3. Use `*Async` request methods so your loop can run other tasks.
4. Await responses directly rather than polling.
5. Disconnect in `finally` to avoid leaked sessions.

## Variations

- Combine with other tasks via `asyncio.gather`.
- For notebooks, use `util.startLoop()` and sync methods instead of `asyncio.run`.
- Add timeout wrappers with `asyncio.wait_for` around specific awaited requests.

## Gotchas Specific to This Use Case

- Do not mix nested `asyncio.run` calls in notebook/event-loop environments.
- Blocking CPU work in coroutine bodies still starves inbound market data handling.
- `RequestTimeout` does not govern `*Async` methods.
