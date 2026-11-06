# control4-test-harness

A mock Control4 Director, in Python, for testing DriverWorks drivers without a controller.

`c4mock.py` runs each driver's `driver.lua` in its own Lua 5.1 state (via lupa) and provides the
parts of the DriverWorks API that drivers commonly use: real TCP clients, timers, Director's
crypto (with Director's quirks, such as refusing an IV for AES-ECB), proxy notifications,
properties and dynamic property lists, variables, persistence, driver-to-driver messages
(`SendToDevice`, `GetDevicesByC4iName`), UDP servers and `C4:url` routed to a fake web service.

```python
from c4mock import Director

d = Director(web=fake_cloud)                       # fake_cloud(method, url, headers, body) -> (code, body)
bridge = d.add("drivers/bridge/driver.lua", {"Discovery": "On"}, "my_bridge.c4z")
light = d.add("drivers/light/driver.lua", {}, "my_light.c4z")
bridge.init(); light.init()
light.proxy("SET_LEVEL", LEVEL=50)
d.run(3, lambda: light.properties.get("Connection Status") == "Online")
print(light.notifies[-1], bridge.persist)
```

A single driver can also be run on its own with `Harness(path, properties)`.

It tests a driver's logic and protocol, not Director itself: anything Director might do
differently should still be checked on a controller.

## Requirements

```sh
pip install lupa cryptography
```

Used by: [control4-tuya](https://github.com/The1TrueJoe/control4-tuya) (`test/`).
