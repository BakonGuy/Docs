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
| In the editor | In C++ |
|---|---|
| (mph) Miles Per Hour | `MPH` |
| (km/h) Kilometers Per Hour | `KPH` |
| (ft/s) Feet Per Second | `FTPS` |
| (m/s) Meters Per Second | `MPS` |
| (kn) Knots | `KNOTS` |
| Furlong Per Fortnight | `Furlong` |
<!-- /side-by-side -->



## Reading Speed Units for a HUD

`GetSpeedUnitData` returns the abbreviation and conversion factors.

This is what you want for a HUD that displays units, rather than hardcoding "MPH".



## Brake Lock Speed

**Brake Lock Speed** (`10`) is the speed below which the wheels fully lock to hold the vehicle still, while the brake is held at `0.95` or more. It is in your **Speed Units**.

Only wheels with a braking, handbrake or driving role lock. Free-rolling wheels never do.

Set it to `0` to disable the lock.



## Cinematic Playback

Enable this **after** recording a cinematic, not before. When on, it prevents cinematic playback fighting against the vehicle's own logic.

Leave it off while driving or recording, or you will be fighting it instead.

`IsCinematic` returns true only while **Cinematic Playback** is on **and** the vehicle is not player controlled, so a vehicle a player takes over behaves normally again.



## Recording a Vehicle with Take Recorder

Add your wheel meshes manually as children of the wheel components before recording, then record the vehicle with Take Recorder as normal.



## Related Settings in Other Categories

A few vehicle-wide behaviors live in other categories:

- **Allow Passive Mode** and **Passive Tick Gatekeeping** (Tick) — see [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md)
- **Upside Down Angle Threshold** (Physics, Advanced) and the **OnVehicleFlipped** event — see [Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md)
- **Disable Skeletal Collisions** (Physics) — see [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md)



## Driving One Vehicle from Another with Input Host

`SetInputHost(OtherVehicle)` makes a vehicle copy its inputs from another AVS vehicle, on the 25 TPS tick.

Throttle, brake, steering and handbrake are all taken from the host. The follower still runs its own physics, its own gears and its own wheels — it is only the driver input that is shared.

Input Host was built for trailers. See [Hitch / Trailers](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Hitch_Trailers.md).

Pass `nullptr` (or an empty reference in Blueprint) to stop copying.

> Clearing the host does not reset the follower's inputs. Whatever was last copied — throttle, brake, steering, handbrake — stays until something sets it, so zero them yourself when you release the host.

> The host relationship is one way. Setting a host on a vehicle does not make the host follow it back.
