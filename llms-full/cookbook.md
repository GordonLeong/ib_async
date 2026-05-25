# Cookbook

## Basic connect/disconnect
**Task**: Start and end a normal session.
```python
from ib_async import IB
ib = IB(); ib.connect('127.0.0.1', 7497, clientId=1)
ib.disconnect()
```
Use unique `clientId` per concurrent session.

## Notebook event loop setup
**Task**: Use in Jupyter.
```python
from ib_async import util
util.startLoop()
```
Call before creating `IB()` in notebooks.

## Enable delayed data
**Task**: Use delayed feed when no real-time subscription.
```python
ib.reqMarketDataType(3)
```
Use `4` for delayed frozen data.

## Convert bars to DataFrame
**Task**: Get pandas output from historical bars.
```python
from ib_async import util
df = util.df(bars)
```
Requires pandas installed.

## Get managed accounts
**Task**: Fetch account list.
```python
accounts = ib.managedAccounts()
print(accounts)
```
Useful before account-specific requests.

## Async connect
**Task**: Connect inside asyncio app.
```python
await ib.connectAsync('127.0.0.1', 7497, clientId=5)
```
Do not wrap this in blocking code.

## What-if order check
**Task**: Ask IBKR for estimated commission/margin.
```python
state = await ib.whatIfOrderAsync(contract, order)
print(state.commission)
```
Good pre-trade validation step.

## Scanner subscription stream
**Task**: Receive scanner updates.
```python
sub = ScannerSubscription(instrument='FUT.US', locationCode='FUT.GLOBEX', scanCode='TOP_PERC_GAIN')
scan = ib.reqScannerSubscription(sub)
scan.updateEvent += lambda rows: print(len(rows))
```
Cancel when done to release subscription.

## News provider codes
**Task**: Build provider code string.
```python
providers = ib.reqNewsProviders()
codes = '+'.join(p.code for p in providers)
```
Use with `reqHistoricalNews`.
