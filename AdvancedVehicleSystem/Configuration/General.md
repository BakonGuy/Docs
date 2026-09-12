# General Vehicle Settings

Vehicle-wide settings under **Advanced Vehicle System → General**. Select `ClassName(self)` in the components list, or press **Class Defaults**, to find them.



## Basic Understanding

These are settings that apply to the vehicle as a whole rather than to a component. Most projects set **Speed Units** once and never touch the rest.

A few vehicle-wide behaviors live in other categories, listed at the bottom of this page.



## Speed Units

**Speed Units** sets the unit for every speed value AVS calculates or returns. It is the conversion between Unreal units (cm/s) and what you want to work in, so every "Speed" output is in this unit.

<!-- side-by-side:57 -->
**Gear speeds and shift points use this unit too.** Changing it after tuning a vehicle does not convert the gear table. The numbers stay the same and now mean something else: a vehicle that topped out at 75 MPH will top out at 75 KPH.

Set it once, before building your gear table, and leave it alone.
<!-- split -->
| Option | Unit |
|---|---|
| `MPH` | Miles per hour |
| `KPH` | Kilometers per hour |
| `FTPS` | Feet per second |
| `MPS` | Meters per second |
| `KNOTS` | Knots |
| `Furlong` | Furlongs per fortnight |
<!-- /side-by-side -->



## Reading Speed Units for a HUD

`GetSpeedUnitData` returns the abbreviation and conversion factors.

This is what you want for a HUD that displays units, rather than hardcoding "MPH".



## Brake Lock Speed

When the brakes are fully engaged, this is the speed below which the wheels fully lock to hold the vehicle still. Default `10.0`.

Without it, a vehicle on a slope creeps downhill under full brakes. Set it to `0.0` to disable the behavior, for example on a vehicle whose brakes have failed.



## Cinematic Playback

Enable this **after** recording a cinematic, not before. When on, it prevents cinematic playback fighting against the vehicle's own logic.

Leave it off while driving or recording, or you will be fighting it instead.



## Related Settings in Other Categories

A few vehicle-wide behaviors live in other categories:

- **Allow Passive Mode** and **Passive Tick Gatekeeping** (Tick) — see [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md)
- **Upside Down Angle Threshold** (Physics, Advanced) and the **OnVehicleFlipped** event — see [Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md)
- **Disable Skeletal Collisions** (Physics) — see [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md)
