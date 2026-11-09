# Camera Pivot

`AVS_CameraPivot` is a component your camera attaches to. It handles look input, keeping the horizon level, and camera movement such as lag and G-force lean. This page builds a third person chase camera for the car from the [Quick Start](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md), then shows setups for other kinds of game.

Every setting is listed in [Camera Pivot Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Camera_Pivot_Settings.md).



## How the Camera Pivot Works

The pivot keeps its own look rotation, separate from the controller's control rotation. Look input goes to the pivot with `AddYawInput` and `AddPitchInput`.

Every frame, after physics, the pivot moves to its place on the vehicle, applies the look rotation, then applies its movement mode. The camera, or a spring arm holding the camera, is attached to the pivot and moves with it.

Because the pivot updates after physics, it follows the vehicle's final position for the frame, without depending on tick order.

A spring arm alone is enough for a fixed chase camera. The pivot adds look input measured from the vehicle, axis locks, lag and G-force movement. The two work together, with the spring arm attached to the pivot.



## Setting Up a Chase Camera

<!-- side-by-side:57 -->
1. Add an `AVS_CameraPivot` component under `VehicleMesh`, at the point the camera should rotate around. For a chase camera, that is the middle of the car, at about roof height.
2. Add a **Spring Arm** under the pivot, and a **Camera** under the spring arm.
3. Turn on **Lock Roll** on the pivot, so the horizon stays level when the car rolls.
4. Send look input to the pivot, as in the code below.
5. Call `ResetPivotRotation` on the pivot when the vehicle is possessed.
<!-- split -->
![Component hierarchy with a spring arm parented under the camera pivot and a camera under the arm](../Assets/Images/_placeholder.png "Pivot, then spring arm, then camera")
<!-- /side-by-side -->

```cpp
// Look input goes to the pivot, not to AddControllerYawInput.
CameraPivot->AddYawInput(LookAxis.X);
CameraPivot->AddPitchInput(LookAxis.Y);
```

The same functions are available in Blueprint on the pivot component.

The pivot finds its camera on BeginPlay by searching its children, including through the spring arm.

Step 5 is needed for the pivot's own rotation, such as a pitch looking down at the car. With **Relative To Actor** on (the default), that rotation is only applied by `ResetPivotRotation`. Calling it on possession also faces a new driver forward.



## Choosing a Movement Mode

Location and rotation each have their own mode: **Location Mode** and **Rotation Mode**. Both start at `None (Static Location)`, which keeps the pivot fixed to the vehicle.

<!-- side-by-side:50 -->
**Lag (Trailing)**

The pivot follows its target smoothly, falling behind while the car moves and catching up when it settles. It never overshoots.

**Location Lag Speed** and **Rotation Lag Speed** set how fast it catches up.
<!-- split -->
**Bounce (G-Force)**

The pivot is on a spring pushed by the car's acceleration. It moves back as the car accelerates, forward as it brakes, and bounces over bumps before settling.

**Stiffness** sets how hard the spring pulls back. **Damping** sets how quickly it stops bouncing.
<!-- /side-by-side -->

The chase camera uses `Lag` for both.



## Tuning the Chase Camera

Set these on the pivot:

- **Location Mode** `Lag (Trailing)`, **Location Lag Speed** `15`
- **Max Location Offset** `60, 60, 30`
- **Rotation Mode** `Lag (Trailing)`, **Rotation Lag Speed** `12`
- **Max Rotation Offset** `5, 10, 25`

The lag speeds set how quickly the camera catches up. Higher is tighter.

**Max Location Offset** (cm) and **Max Rotation Offset** (degrees, as roll, pitch, yaw) set how far the camera can fall behind before it stops. With small offsets, the camera only swings a little through corners.

Raising the offsets and lowering the lag speeds together gives a looser camera that swings further into corners. Raising only the offsets lets the camera fall further behind but recover at the same speed. Lowering only the lag speeds leaves it sitting at the offset limit.

These values are a starting point. They have not been tested in a shipped game.



## Limiting Look Input

Turn on **Limit Rotation** to clamp how far the player can look. **Input Rotation Limits** sets the limit in degrees each way, as roll, pitch and yaw. The default `900, 65, 135` leaves roll unlimited, limits pitch to 65 degrees and yaw to 135. A value of `180` or more leaves that axis unlimited.

Without a pitch limit, players can look straight up or down and lose sight of the car.

**Lock Roll**, **Lock Pitch** and **Lock Yaw** hold that axis at `0` in world space, whatever the car is doing. With the locks off, the camera tilts with the car, as a cockpit camera does.



## Example Setups for Other Games

The cockpit and cinematic values are untested starting points. The off-road setups are used in a real game with a heavy, grounded feel.

### Simulation Cockpit Camera

A camera inside the car moves only a few centimeters, so the horizon stays readable. A stiff, well damped spring recovers quickly. **Location Spring Scale** gives full movement forward and back, and half up and down and side to side. The locks are off, so the view tilts with the car.

- **Location Mode** `Bounce (G-Force)`, **Location Stiffness** `300`, **Location Damping** `20`
- **Max Location Offset** `5, 5, 5`
- **Location Spring Scale** `1.0, 0.5, 0.5`
- **Rotation Mode** `Bounce (G-Force)`, **Max Rotation Offset** `3, 3, 3`
- **Limit Rotation** on, all locks off

### Cinematic or Replay Camera

Slow lag with large offsets lets the camera drift well off the car before it catches up. `Lag` keeps the drift steady, where `Bounce` would react to each bump. Put it on its own pivot and camera, and switch to it as shown in Switching Between Two Cameras.

- **Location Mode** `Lag (Trailing)`, **Location Lag Speed** `3`
- **Max Location Offset** `400, 400, 200`
- **Rotation Mode** `Lag (Trailing)`, **Rotation Lag Speed** `3`
- **Max Rotation Offset** `15, 20, 90`
- **Lock Roll** on

### Off-Road Chase Camera

Rough terrain and collisions throw this camera around, and it snaps back like whiplash while staying usable at high speed. A very low **Location Stiffness** and a very large **Max Location Offset** let it swing far. **Location Damping** of `15` against that low stiffness brings it back quickly. Rotation uses `Lag`, so the view turns smoothly with the vehicle instead of bouncing.

```cpp
Pivot_ThirdPersonCam = CreateDefaultSubobject<UAVS_CameraPivot>(TEXT("Pivot_ThirdPersonCam"));
Pivot_ThirdPersonCam->SetupAttachment(VehicleMesh);

Pivot_ThirdPersonCam->LocationMode = EPivotMovementMode::Bounce;
Pivot_ThirdPersonCam->LocationStiffness = 25.0f;
Pivot_ThirdPersonCam->LocationDamping = 15.0f;
Pivot_ThirdPersonCam->MaxLocationOffset = FVector(2000.0);

Pivot_ThirdPersonCam->RotationMode = EPivotMovementMode::Lag;
Pivot_ThirdPersonCam->MaxRotationOffset = FVector(170.0);

Pivot_ThirdPersonCam->SetRelativeLocation(FVector(-39.5, 0.0, 108.0));
Pivot_ThirdPersonCam->SetRelativeRotation(FRotator(-10.0, 0.0, 0.0)); // Looks slightly down

CameraArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraArm"));
CameraArm->SetupAttachment(Pivot_ThirdPersonCam);
CameraArm->TargetArmLength = 420.0f;

ThirdPersonCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("ThirdPersonCamera"));
ThirdPersonCamera->SetupAttachment(CameraArm);
ThirdPersonCamera->SetAutoActivate(false);
```

### Off-Road First Person Camera

The camera is in the cab. It uses `Bounce` with a stiff spring, and **Location Spring Scale** gives mostly forward and back movement, a little up and down, and very little side to side. **Relative To Actor** is off, so look input is measured from the pivot's own rotation, and **Limit Rotation** keeps the player looking roughly forward.

```cpp
Pivot_FirstPersonCam = CreateDefaultSubobject<UAVS_CameraPivot>(TEXT("Pivot_FirstPersonCam"));
Pivot_FirstPersonCam->SetupAttachment(VehicleMesh);
Pivot_FirstPersonCam->bRelativeToActor = false;
Pivot_FirstPersonCam->bLimitRotation = true;

Pivot_FirstPersonCam->LocationMode = EPivotMovementMode::Bounce;
Pivot_FirstPersonCam->LocationStiffness = 200.0f;
Pivot_FirstPersonCam->LocationDamping = 15.0f;
Pivot_FirstPersonCam->LocationSpringScale = FVector(0.75, 0.25, 0.4);

Pivot_FirstPersonCam->RotationMode = EPivotMovementMode::Bounce;
Pivot_FirstPersonCam->MaxRotationOffset = FVector(15.0, 15.0, 15.0);

Pivot_FirstPersonCam->SetRelativeLocation(FVector(-30.0, -40.0, 110.0));

FirstPersonCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("FirstPersonCamera"));
FirstPersonCamera->SetupAttachment(Pivot_FirstPersonCam);
FirstPersonCamera->SetAutoActivate(false);
```

With **Relative To Actor** off, keep the pivot's own rotation at zero, as this example does. `ResetPivotRotation` would otherwise apply it twice.



## Switching Between Two Cameras

Give each camera its own pivot. With **Detect Active Camera** on (the default), a pivot only updates while its camera is active, so switching is just activating one camera and deactivating the other.

```cpp
void AMyVehicle::UpdateCameraPerspective(const bool bFirstPerson)
{
	FirstPersonCamera->SetActive(bFirstPerson);
	ThirdPersonCamera->SetActive(!bFirstPerson);
}
```

Reset both pivots when the controller changes, so a new driver starts looking forward:

```cpp
void AMyVehicle::NotifyControllerChanged()
{
	Super::NotifyControllerChanged();
	Pivot_FirstPersonCam->ResetPivotRotation();
	Pivot_ThirdPersonCam->ResetPivotRotation();
}
```

If one pivot has more than one camera under it, call `SetTrackedCamera` to choose which one it watches.



## Fixing Camera Pivot Problems

### Camera Does Not Move or Respond to Look Input

With **Detect Active Camera** on, the pivot only updates while it has an active camera beneath it. Check that a camera is attached under the pivot and is active, or turn **Detect Active Camera** off.

### Camera Starts Level Instead of at the Pivot's Angle

With **Relative To Actor** on, the pivot's own rotation is applied by `ResetPivotRotation`. Call it when the vehicle is possessed.



## Next Steps

- [Camera Pivot Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Camera_Pivot_Settings.md): every setting and function.
- [Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md): how the vehicle body moves, which the camera follows.
