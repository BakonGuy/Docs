# Drivetrain and Physics Overrides

New in 1.5. Two override points on the physics thread let you replace AVS's drivetrain and force logic with your own.

These are **C++ only** and run on the physics thread. Mistakes here produce vehicle behavior that is difficult to trace. Tune the existing settings first.



## Basic Understanding

There are two override points, and they sit at different stages of the same physics tick.

**`PhysicsTickDrivetrain`** runs first, once per vehicle per substep, before the wheel loop. It decides how much torque each wheel gets. This is the one to use for a differential, a torque split, or an electric motor curve.

**`AVS_ApplyPhysicsTick`** runs last. It receives the finished, ordered list of forces AVS is about to apply, which you can inspect, filter, scale, or add to.

> These exist so you can replace AVS behavior without forking the plugin. Use them when the stock drivetrain cannot represent your vehicle.



## Physics Thread Rules

Both override points run on the physics thread, not the game thread. Before writing anything:

- Only access state owned by the physics thread.
- Do not touch actors, components, Blueprints, or anything game-thread.
- Do not allocate or log heavily. This runs per substep.

> Reading a UPROPERTY set on the game thread appears to work, then produces torn values under load. Cache what you need into the physics input instead.



## Reading the Physics Force List

<!-- side-by-side:57 -->
**Override `AVS_ApplyPhysicsTick` first and just read.** Do not modify anything on the first pass.

Logging the force list for a vehicle you already understand shows exactly what AVS applies, in order. It will usually tell you whether the drivetrain override is needed at all.

Call the parent implementation unless you intend to replace force application entirely.
<!-- split -->
![Console output showing the ordered physics force list logged from AVS_ApplyPhysicsTick](../Assets/Images/_placeholder.png "Read the force list before changing anything")
<!-- /side-by-side -->



## What a Physics Force Carries

Each `FAVS_PhysicsForce` carries:

- a mode
- a location, in cm
- a force, in centinewtons
- an optional brake torque, in Nm
- the delta time
- an acceleration-change flag
- the originating wheel index



## PhysicsTickDrivetrain Override

```cpp
virtual void PhysicsTickDrivetrain(
    float StepDeltaTime,
    const TArray<FAVS1_Wheel_Config>& InWheels,
    const TArray<FAVS1_Wheel_State>& InWheelStates,
    FAVS_Inputs& VehicleInputs,
    TArray<FAVS_WheelDrivetrainOutput>& DrivetrainWheelOutputs)
```

Called once per vehicle per Chaos substep, before the wheel loop.

`InWheels`, `InWheelStates`, and `DrivetrainWheelOutputs` are all in the **same order**, and `InWheelStates` holds persistent state at the start of this substep.

This is where a real differential, a torque split AVS does not model, an electric motor curve, or per-wheel traction control belongs.

> **Do not resize or reorder the arrays.** Per-wheel outputs are consumed by raycast wheels.



## Wheel Drivetrain Output

Each entry in `DrivetrainWheelOutputs` carries two independent overrides:

```cpp
struct FAVS_WheelDrivetrainOutput
{
	float DriveTorque = 0.0f;       // [hNm] signed torque around the wheel axis
	bool  bOverrideDriveTorque = false;

	float AngularVelocity = 0.0f;   // [rad/s] preserved instead of derived from ground speed
	bool  bOverrideAngularVelocity = false;
};
```

Wheels you do not flag keep stock behavior, so an override can cover one axle and leave the rest of the vehicle unchanged.



## Example: Open Differential

A simple open differential that sends torque to whichever driven wheel has more grip.

```cpp
void AMyVehicle::PhysicsTickDrivetrain(
	float StepDeltaTime,
	const TArray<FAVS1_Wheel_Config>& InWheels,
	const TArray<FAVS1_Wheel_State>& InWheelStates,
	FAVS_Inputs& VehicleInputs,
	TArray<FAVS_WheelDrivetrainOutput>& DrivetrainWheelOutputs)
{
	// Arrays are index-matched. Never resize or reorder them.
	for( int32 i = 0; i < InWheels.Num(); ++i )
	{
		if( !InWheels[i].IsDrivingWheel ) continue;

		// Slip above 1.0 means this wheel is asking for more grip than it has.
		const float Slip = FMath::Abs(InWheelStates[i].Slip2D.X);
		const float GripScale = FMath::Clamp(1.0f - Slip, 0.1f, 1.0f);

		DrivetrainWheelOutputs[i].DriveTorque = VehicleInputs.Torque * GripScale;
		DrivetrainWheelOutputs[i].bOverrideDriveTorque = true;
	}
}
```



## AVS_ApplyPhysicsTick Override

```cpp
virtual void AVS_ApplyPhysicsTick(FAVS_PhysicsForces& InPhysicsForces)
```

Applies the ordered force operations calculated during the physics tick. Overriding it lets you inspect, modify, filter, or add to every force AVS is about to apply.

Filtering is the safest use: dropping or scaling specific forces under conditions you control.

Adding your own forces is also straightforward. Replacing the whole set means reimplementing suspension and traction.
