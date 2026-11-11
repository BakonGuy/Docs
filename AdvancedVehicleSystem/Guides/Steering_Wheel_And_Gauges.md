# Animating a Steering Wheel and Gauges

Turning a steering wheel mesh, and moving speedometer and tachometer needles, from the vehicle's own values. AVS has no component for this. Each part is a mesh on the vehicle that your Blueprint rotates.

> The examples use a separate Static Mesh component for each part because that is the simplest way to learn the steps. Many individual components cost more than they need to, especially with many vehicles. Animating the parts as bones of a cosmetic skeletal mesh, or in a material driven by parameters from these values, is more efficient. The values to read are the same either way.



## Values to Read

- `GetCurrentSteering` returns the steering the wheels are using, from `-1` (full left) to `1` (full right), after the falloff curve and smoothing. It updates every frame.
- `GetAirSpeed` returns the vehicle's speed in its **Speed Units**. It is measured from the body's whole velocity, so it is never negative and includes sliding sideways. It updates 25 times a second.
- `GetEngineRPM` returns the engine RPM. It updates 25 times a second.
- `GetSpeedUnitData` returns the selected unit and its abbreviation, such as `mph`, for a digital readout.

Steering inputs run ahead of `GetCurrentSteering` while the smoothing catches up. Use `GetCurrentSteering` so the steering wheel matches the road wheels. See [Steering](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Steering.md#reading-steering-values).



## Turning the Steering Wheel

1. Add a Scene component under `VehicleMesh` at the center of the steering wheel, rotated to match the steering column's angle.
2. Add the steering wheel as a Static Mesh component under it, with no rotation of its own. The mesh's pivot must be at the center of the wheel.
3. On Tick, set the steering wheel's relative rotation to a roll of `GetCurrentSteering` multiplied by the steering wheel's angle at full lock.

The Scene component holds the column's tilt, so the steering wheel only ever needs a roll. A road car's steering wheel turns about `450` degrees each way. Use the value that fits your vehicle. If the wheel turns the wrong way, negate the angle.

`GetCurrentSteering` updates every frame, so the steering wheel needs no smoothing.



## Moving Gauge Needles

1. Add each needle as a Static Mesh component, with its pivot at the point it rotates around.
2. Map the value to an angle with **Map Range Clamped**. For a speedometer that reads `0` to `160`, with a needle that sweeps `270` degrees, map `GetAirSpeed` from `0`–`160` to `0`–`270`.
3. On Tick, move the needle's current angle toward that target with **FInterp To**, and set its relative rotation from the result.

Speed and RPM only change 25 times a second. A needle set straight from them moves in visible steps at higher frame rates. **FInterp To** smooths the steps out. A higher **Interp Speed** makes the needle follow more closely.

The same steps work for a tachometer with `GetEngineRPM`. The [RPM](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Engine_And_Transmission.md#rpm) section covers how RPM moves in and out of gear.

The same logic in C++, as a function on a vehicle class derived from `AAVS_Vehicle` and called from Tick:

```cpp
void AMyVehicle::UpdateDashboard(float DeltaSeconds)
{
	// Steering wheel: a roll of up to 450 degrees each way.
	SteeringWheelMesh->SetRelativeRotation(FRotator(0.0f, 0.0f, GetCurrentSteering() * 450.0f));

	// Speedometer: 0 to 160 in the vehicle's Speed Units, over a 270 degree sweep.
	const float TargetSpeedAngle = FMath::GetMappedRangeValueClamped(
		FVector2D(0.0f, 160.0f), FVector2D(0.0f, 270.0f), GetAirSpeed());
	SpeedNeedleAngle = FMath::FInterpTo(SpeedNeedleAngle, TargetSpeedAngle, DeltaSeconds, 10.0f);
	SpeedNeedleMesh->SetRelativeRotation(FRotator(0.0f, 0.0f, SpeedNeedleAngle));

	// Tachometer: 0 to 8000 RPM over the same sweep.
	const float TargetRpmAngle = FMath::GetMappedRangeValueClamped(
		FVector2D(0.0f, 8000.0f), FVector2D(0.0f, 270.0f), GetEngineRPM());
	RpmNeedleAngle = FMath::FInterpTo(RpmNeedleAngle, TargetRpmAngle, DeltaSeconds, 10.0f);
	RpmNeedleMesh->SetRelativeRotation(FRotator(0.0f, 0.0f, RpmNeedleAngle));
}
```

`SteeringWheelMesh`, `SpeedNeedleMesh` and `RpmNeedleMesh` are your components. `SpeedNeedleAngle` and `RpmNeedleAngle` are float members that keep each needle's angle between frames.



## Digital Speed Readout

Round `GetAirSpeed` to a whole number, and add the **Abbreviation** from `GetSpeedUnitData`. The readout stays correct if **Speed Units** changes. See [Speed Units](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/General.md#speed-units).

For a gear indicator, see [Shifter Position and Current Gear](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Engine_And_Transmission.md#shifter-position-and-current-gear).



## Next Steps

- [Steering](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Steering.md): the falloff curve and smoothing behind `GetCurrentSteering`.
- [Engine and Transmission](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Engine_And_Transmission.md): RPM and gears.
- [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md): which tick event to update from.
