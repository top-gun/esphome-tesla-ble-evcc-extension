# esphome-tesla-ble-evcc-extension
Adds additional helper entities to esphome-tesla-ble that fully support the EVCC logic

## Topology flags

The package supports two ways of wiring evcc, selectable via substitutions:

```yaml
substitutions:
  stop_charge_on_unplug: "true"   # default
  start_charge_on_plugin: "false" # default
```

- **evcc controls the car** (dumb wallbox, e.g. Tesla Wall Connector: evcc talks to `Charger EVCC` / `Charging Amps EVCC`): keep the defaults. Unplugging turns the charger switch off so a stale charge enable does not linger.
- **evcc controls a smart wallbox** (PWM current control, the car is only a data source): set `stop_charge_on_unplug: "false"`. Otherwise the stop command sent on unplug is never undone - evcc never talks to the car in this topology - and the car sits at "Stopped" forever on the next plug-in.

  With `stop_charge_on_unplug: "false"` the car is never stopped over BLE, so it resumes charging on its own when the wallbox offers current (normal EV behaviour) and no start command is needed. `start_charge_on_plugin: "false"` (the default) is therefore the recommended setting for this topology.

  `start_charge_on_plugin: "true"` is only a one-time migration aid: if a car is already stuck "Stopped" from a previous version, it re-sends a charge enable on the next plug-in (ISO state A -> B) to clear it. Do not leave it on permanently - it fires on every A -> B transition (including pilot glitches and reboots), can override a Tesla-side charge schedule, and may fail with `no_power` if evcc is not yet offering current. Prefer clearing a stuck state once from the Tesla app instead.
