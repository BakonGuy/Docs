# Driving a Vehicle with AI

AVS has no built-in AI or path following. An AI-driven vehicle uses the same input functions as a player's, called by an AI controller. This page sets up the controller, steers toward a target, and follows a spline.

> The examples on this page explain how an AI drives an AVS vehicle. They are not production-ready AI and will not give production-quality results. They do not brake for corners, avoid obstacles or other vehicles, or recover when stuck. Use them as a starting point for your own AI.



## Possessing the Vehicle with an AI Controller

1. On the vehicle, set **Auto Possess AI** to `Placed in World or Spawned`.
2. Set **AI Controller Class** to your AI controller.
3. Start the engine and shift into Drive, as for a player. See [Step 3 of the Quick Start](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md#step-3-start-the-engine-on-beginplay).

A vehicle possessed by an AI controller never goes passive, so it keeps simulating while the AI drives it. See [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md).

In multiplayer, AI controllers exist only on the server. When no player controls a vehicle, the server counts as its owner, so input functions called there work and replicate to clients. See [Calling Input Functions in Multiplayer](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Networking.md#calling-input-functions-in-multiplayer).



## Steering Toward a Target

Convert the target's location into the vehicle's local space. X is forward and Y is right, for a vehicle built facing X (see [Preparing Vehicle Meshes](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Preparing_Vehicle_Meshes.md)). The angle between them gives the steering. Positive steering turns right.

In Blueprint, use **Inverse Transform Location** with the vehicle's transform, then **Atan2 (Degrees)** with the local Y and X. Divide the angle by the angle that should give full steering, clamp it to `-1`–`1`, and pass it to **Set Steering Input**.

The same in C++, on an AI controller:

```cpp
void AMyVehicleAIController::DriveToward(const FVector& Target, float TargetSpeed)
{
	AAVS_Vehicle* Vehicle = Cast<AAVS_Vehicle>(GetPawn());
	if (!Vehicle) return;

	// Target in the vehicle's local space: X is forward, Y is right.
	const FVector Local = Vehicle->GetActorTransform().InverseTransformPosition(Target);
	const float AngleToTarget = FMath::RadiansToDegrees(FMath::Atan2(Local.Y, Local.X));

	// Full steering when the target is 30 degrees or more off to one side.
	Vehicle->SetSteeringInput(FMath::Clamp(AngleToTarget / 30.0f, -1.0f, 1.0f));

	// Throttle until TargetSpeed, in the vehicle's Speed Units, then coast.
	Vehicle->SetThrottleInput(Vehicle->GetAirSpeed() < TargetSpeed ? 1.0f : 0.0f);
}
```

Call it from the AI controller's Tick. A smaller full-steering angle turns in harder and can make the vehicle weave. A larger one turns in later.

If the AI also brakes, check the transmission mode. With **Automatic Shifter Positon** on, pressing the brake from a stop selects Reverse. With it off and the shifter in Drive, the brake only slows the vehicle. See [Choosing a Transmission Mode](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Engine_And_Transmission.md#choosing-a-transmission-mode).



## Following a Spline

To follow a path, steer toward a point a fixed distance ahead along a spline:

```cpp
// Spline is a USplineComponent on an actor placed along the route.
const float Key = Spline->FindInputKeyClosestToWorldLocation(Vehicle->GetActorLocation());
const float Distance = Spline->GetDistanceAlongSplineAtSplineInputKey(Key);

// 1500 cm ahead. A closed loop wraps around to the start.
const float AheadDistance = FMath::Fmod(Distance + 1500.0f, Spline->GetSplineLength());
const FVector AheadPoint = Spline->GetLocationAtDistanceAlongSpline(AheadDistance, ESplineCoordinateSpace::World);

DriveToward(AheadPoint, 40.0f);
```

A point further ahead cuts corners and steers more smoothly. A closer point follows the spline more tightly. To slow down before tight corners, lower the target speed when the spline turns sharply ahead, for example by comparing the spline's direction at the vehicle with its direction at the point ahead.



## Next Steps

- [Networking](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Networking.md): how inputs replicate.
- [Steering](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Steering.md): the falloff curve that limits steering at speed.
- [Engine and Transmission](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Engine_And_Transmission.md): transmission modes and gears.
