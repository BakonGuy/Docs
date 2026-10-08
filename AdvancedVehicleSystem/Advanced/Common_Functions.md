# Common Functions

`AVS_CommonFunctions` is a Blueprint function library that ships with the plugin. AVS uses it internally, and the functions are available in any Blueprint.

Nothing here is required to use AVS. They are exposed because they are the same helpers the plugin uses, and may be useful.



## Math Helpers

| Function | Does |
|---|---|
| `GetPercentageInRange(Value, RangeBegin, RangeEnd)` | Where a value sits through a range — `0` at the beginning, `1` at the end. Not clamped, so values outside the range return below `0` or above `1`. |
| `LinearSpeedToRads(cm_per_sec, Radius)` | Converts linear speed in cm/s to angular speed in rad/s, for a wheel of that radius in cm. |
| `GetWheelInertia(Target, MassKg, RadiusCm)` | Rotational inertia in kg·m², as `0.4 × mass × radius²`. The component argument is not used. |
| `FastDist(A, B)` | Manhattan distance — the sum of the differences on each axis. Faster than a true distance, and less accurate. |
| `SetFloatPrecision(Value, Precision)` | Rounds a float to a number of decimal places, from `0` to `6`. |
| `IsTowardZero(Old, New)` | True if `New` is closer to zero than `Old`. |

`LinearSpeedToRads` and `GetWheelInertia` are worth knowing if you are writing a [drivetrain override](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Physics_Overrides.md), since they match what AVS uses internally.



## Physics Helpers

`GetMeshRadius` and `GetMeshDiameter` measure a component's bounds. For a static mesh, the radius is half the mesh's height, scaled. This is the same call AVS uses for **Auto Wheel Radius**, so a wheel you size yourself matches one AVS sized for you.

`GetMeshCenterOfMass` returns a body's center of mass, relative to the component.

`SetLinearDamping` and `SetAngularDamping` set damping on a primitive component, or on one bone of it if a bone name is given.



## Debug Helpers

`PrintToScreenWithTag` prints to the screen, replacing the previous message that used the same **Tag**. A normal print string adds a new line every call; tagging updates one line in place, which suits values printed every frame.

`GetUnrealEngineVersion` outputs the running engine version as **Major**, **Minor** and **Patch** numbers.

`GetAvsPluginVersion` returns the installed AVS version as a string. It is expensive, so do not call it every frame.
