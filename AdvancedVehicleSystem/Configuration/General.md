# General Vehicle Settings

The settings under **Advanced Vehicle System → General**, which apply to the whole vehicle. To find them, select `ClassName(self)` in the components list or press **Class Defaults**.

Every vehicle setting is listed in [Vehicle Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Vehicle_Settings.md).



## Speed Units

**Speed Units** (`MPH`) is the unit for every speed AVS takes or returns. That includes `GetAirSpeed`, the steering falloff curve and every speed in the gear table.

Set it before building the gear table. Changing it later does not convert the numbers already entered. A vehicle with a top gear ending at `75` mph would end at `75` km/h after switching to km/h.

The options are `(mph) Miles Per Hour`, `(km/h) Kilometers Per Hour`, `(ft/s) Feet Per Second`, `(m/s) Meters Per Second`, `(kn) Knots` and `Furlong Per Fortnight`.

`GetSpeedUnitData` returns the selected unit, its abbreviation (such as `mph`), and the factors for converting to and from cm/s. A speedometer that shows the abbreviation stays correct when **Speed Units** changes.



## Brake Lock Speed

Below **Brake Lock Speed** (`10`), with the brake held at `0.95` or more, the wheels lock to hold the vehicle still.

Only wheels with **Is Braking Wheel**, **Is Handbrake Wheel** or **Is Driving Wheel** on lock. `0` turns the lock off.



## Cinematic Playback

Turn on **Cinematic Playback** after recording a vehicle for a cinematic.

While it is on and no player controls the vehicle, AVS releases every wheel lock (handbrake, Park and **Brake Lock Speed**), so the recorded wheel motion plays back. `IsCinematic` returns this state. A vehicle a player takes over behaves normally.

To record a vehicle with Take Recorder, first add the wheel meshes yourself as children of the wheel components, then record as normal.



## Next Steps

- [Engine and Transmission](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Engine_And_Transmission.md): building the gear table.
- [Steering](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Steering.md): steering speed and the falloff curve.
- [Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md): mass, center of mass and moving a vehicle.
