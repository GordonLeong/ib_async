# Antipatterns & Gotchas

## Blocking with `time.sleep`
**What you might do**:
```python
import time
time.sleep(5)
```
**What goes wrong**: Incoming IB messages are not processed; state updates lag or stall.
**Do this instead**:
```python
ib.sleep(5)
```
**Why**: `ib.sleep` yields to the internal event loop.

## Closing immediately after order burst
**What you might do**:
```python
ib.placeOrder(contract, order)
ib.disconnect()
```
**What goes wrong**: Unsent socket data may be dropped on short-lived sessions.
**Do this instead**:
```python
ib.placeOrder(contract, order)
ib.sleep(1)
ib.disconnect()
```
**Why**: Allows protocol buffers to flush.

## Assuming failed requests always raise
**What you might do**:
```python
result = await ib.whatIfOrderAsync(bad_contract, order)
# assume exception was raised if invalid
```
**What goes wrong**: Default behavior may return empty result instead of raising.
**Do this instead**:
```python
ib.RaiseRequestErrors = True
```
**Why**: Enables explicit `RequestError` exceptions.
