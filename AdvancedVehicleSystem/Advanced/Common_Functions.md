# Common Functions

`AVS_CommonFunctions` is a Blueprint function library that ships with the plugin. AVS uses it internally, and the functions are available in any Blueprint.

Nothing here is required to use AVS. They are exposed because they are the same helpers the plugin uses, and may be useful.



## Math Helpers

| Function | Does |
|---|---|
| `GetPercentageInRange(Min, Max, Value)` | Where a value sits between two bounds, as 0-1 |
| `LinearSpeedToRads(Speed, Radius)` | Converts linear speed to angular speed for a wheel of that radius |
| `GetWheelInertia(Mass, Radius)` | Rotational inertia of a wheel |
| `FastDist(A, B)` | Distance check that skips the square root, for comparisons |
| `SetFloatPrecision(Value, Precision)` | Rounds a float to a number of decimal places |

`LinearSpeedToRads` and `GetWheelInertia` are worth knowing if you are writing a [drivetrain override](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Physics_Overrides.md), since they match what AVS uses internally.



## Physics Helpers

`GetMeshRadius` and `GetMeshDiameter` measure a mesh's bounds. `SetAngularDamping` sets angular damping on a primitive component.

`GetMeshRadius` is the same call AVS uses for **Auto Wheel Radius**, so a wheel you size yourself matches one AVS sized for you.



## Debug Helpers

`PrintToScreenWithTag` prints to the screen, replacing the previous line that used the same tag. `GetUnrealEngineVersion` returns the running engine version as a string.

`PrintToScreenWithTag` is the one to use for per-frame values. A normal print string pushes a new line every frame and scrolls the screen; tagging updates one line in place.
