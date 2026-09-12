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

`IsCinematic` returns true only while **Cinematic Playback** is on **and** the vehicle is not player controlled, so a vehicle a player takes over behaves normally again.



## Recording a Vehicle with Take Recorder

Wheel meshes have to exist as real components to be recorded, and AVS creates them at runtime by default.

Add your wheel meshes manually as children of the wheel components before recording, then record the vehicle with Take Recorder as normal.



## Related Settings in Other Categories

A few vehicle-wide behaviors live in other categories:

- **Allow Passive Mode** and **Passive Tick Gatekeeping** (Tick) — see [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md)
- **Upside Down Angle Threshold** (Physics, Advanced) and the **OnVehicleFlipped** event — see [Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md)
- **Disable Skeletal Collisions** (Physics) — see [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md)



## Driving One Vehicle from Another with Input Host

`SetInputHost(OtherVehicle)` makes a vehicle copy its inputs from another AVS vehicle, on the 25 TPS tick.

Throttle, brake, steering and handbrake are all taken from the host. The follower still runs its own physics, its own gears and its own wheels — it is only the driver input that is shared.

This is the supported way to build a vehicle that is really several AVS vehicles moving as one: an articulated bus, a road train, a powered trailer that has to brake with the truck.

Pass `nullptr` (or an empty reference in Blueprint) to release it and return the vehicle to its own input.

> The host relationship is one way. Setting a host on a vehicle does not make the host follow it back.
