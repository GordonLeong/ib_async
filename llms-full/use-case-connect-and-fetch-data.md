# Use Case: Connect and Fetch Market Data

## What You're Trying to Do

You need a working IBKR session and want both live quotes and historical bars for one symbol.

## Complete Working Example

```python
from ib_async import IB, Stock, util

ib = IB()
ib.connect("127.0.0.1", 7497, clientId=7)

try:
    contract = Stock("MSFT", "SMART", "USD")
    ib.qualifyContracts(contract)

    ticker = ib.reqMktData(contract, "", False, False)
    ib.sleep(2)
    print("Live quote", {"last": ticker.last, "bid": ticker.bid, "ask": ticker.ask})

    bars = ib.reqHistoricalData(
        contract,
        endDateTime="",
        durationStr="2 D",
        barSizeSetting="5 mins",
        whatToShow="TRADES",
        useRTH=True,
    )
    df = util.df(bars)
    print(df.tail(3))
finally:
    ib.disconnect()
```

## Step-by-Step Breakdown

1. Create `IB()`; this is the central object for connection, requests, and cached state.
2. `connect()` opens the socket and blocks until API is ready.
3. Create `Stock` contract and call `qualifyContracts` so IBKR resolves identifiers.
4. `reqMktData` starts a subscription and returns a mutable `Ticker` object.
5. `ib.sleep(2)` yields control so network packets can populate ticker fields.
6. `reqHistoricalData` returns bars for a fixed window.
7. `util.df` converts bar objects to a pandas-friendly structure.
8. `disconnect()` closes cleanly and avoids stale sessions.

## Variations

- Async version: swap to `await ib.connectAsync(...)` and `await ib.reqHistoricalDataAsync(...)`.
- Delayed data mode: call `ib.reqMarketDataType(3)` before `reqMktData`.
- Bulk backfill: loop `reqHistoricalData` backward using first bar timestamp from each response.

## Gotchas Specific to This Use Case

- If you use `time.sleep` instead of `ib.sleep`, ticker fields may stay empty because the event loop is blocked.
- Unqualified contracts can cause ambiguous-contract or empty-data responses.
- Missing market-data subscriptions can produce delayed/empty live fields.
