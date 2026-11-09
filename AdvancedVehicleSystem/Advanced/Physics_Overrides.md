# Drivetrain and Physics Overrides

New in 1.5. Two override points on the physics thread let you replace AVS's drivetrain and force logic with your own.

Both are C++ only and run on the physics thread.



## Basic Understanding

There are two override points, and they sit at different stages of the same physics tick.

`PhysicsTickDrivetrain` runs first, once per vehicle per substep, before the wheel loop. It can replace the drive torque or the angular velocity of each raycast wheel, for example for a differential, a torque split or an electric motor curve. Physics mode wheels do not use it.

`AVS_ApplyPhysicsTick` runs last. It receives the finished, ordered list of forces AVS is about to apply, which you can inspect, filter, scale, or add to.



## Physics Thread Rules

Both override points run on the physics thread, not the game thread:

- Only access state owned by the physics thread.
- Do not touch actors, components, Blueprints, or anything else on the game thread.
- Keep the work small. It runs once per substep.



## Reading the Physics Force List

<!-- side-by-side:57 -->
Overriding `AVS_ApplyPhysicsTick` and logging the force list, without changing it, shows exactly what AVS applies and in what order.

The parent implementation applies the forces. Call it unless you intend to replace force application entirely.
<!-- split -->
![Console output showing the ordered physics force list logged from AVS_ApplyPhysicsTick](../Assets/Images/_placeholder.png "Read the force list before changing anything")
<!-- /side-by-side -->



## What a Physics Force Carries

Each `FAVS_PhysicsForce` carries:

- a mode: `Force`, `ForceAtLocation`, `ProxyForceAtLocation` or `Brake`
- what it acts on: a component, or for `ProxyForceAtLocation` a physics proxy
- a location, in cm
- a force, in centinewtons
- a brake torque, in Nm, for `Brake` operations
- the delta time
- an acceleration-change flag
- the originating wheel index

`ProxyForceAtLocation` is how AVS pushes back on whatever a wheel is standing on, while **Wheels Push Physics** is on. When a wheel rests on another physics object, the object receives the opposite of the force the wheel applies: suspension and tire friction for a raycast wheel, and suspension only for a physics wheel, whose friction comes from the physics engine.



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

> Do not resize or reorder the arrays. The outputs are only used by raycast wheels.



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



## Example: Cutting Torque to Slipping Wheels

A simple traction control that scales back each driven wheel's torque as it starts to slip.

`DriveTorque` is signed around the wheel's axis, and AVS does not flip it for you. The example applies the same direction rule as the stock drivetrain, so reverse and **Invert Torque** wheels still work.

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

		// Same direction rule as the stock drivetrain.
		float Torque = VehicleInputs.Torque * GripScale;
		if( InWheels[i].InvertTorque ^ VehicleInputs.ReverseTorque ) Torque *= -1.0f;

		DrivetrainWheelOutputs[i].DriveTorque = Torque; // [hNm]
		DrivetrainWheelOutputs[i].bOverrideDriveTorque = true;
	}
}
```



## AVS_ApplyPhysicsTick Override

```cpp
virtual void AVS_ApplyPhysicsTick(FAVS_PhysicsForces& InPhysicsForces)
```

Applies the ordered force operations calculated during the physics tick. Overriding it lets you inspect, modify, filter, or add to every force AVS is about to apply.

Replacing the whole set means reimplementing suspension and traction.
