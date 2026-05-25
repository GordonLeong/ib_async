# Integration Points

## PyQt / PySide
**Use case**: Keep UI responsive while streaming ticks and making requests.
```python
from ib_async import IB, util

util.patchAsyncio()
util.useQt()  # or util.useQt('PySide6')

ib = IB()
ib.connect('127.0.0.1', 7497, clientId=1)
IB.run()
```
**Notes**: The official example wires `pendingTickersEvent` to table updates.

## Tkinter
**Use case**: Run IB networking alongside Tk event loop.
```python
# See examples/tk.py pattern and docs recipe reference.
```
**Notes**: Use integration helper pattern; avoid separate blocking loops.
