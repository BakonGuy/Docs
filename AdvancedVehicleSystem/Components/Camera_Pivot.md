# Camera Pivot

`AVS_CameraPivot` is the camera rig you would otherwise rebuild on every project — look input, what happens when the vehicle flips, and movement like G-force lean or camera lag.



## Camera Pivot vs Spring Arm

For a fixed chase camera, a spring arm is usually enough. The pivot is for cameras that need to react to the vehicle.

Camera rigs built from control rotation and spring arms often stutter, because they update at the wrong point relative to the vehicle's physics. The pivot ticks in `PostPhysics`, so it always reads the vehicle's final transform for the frame. It also keeps look input separate from control rotation, so it doesn't rely on Unreal's rotation handling.



## Adding a Camera Pivot

<!-- side-by-side:57 -->
**1. Add the pivot** to your vehicle and position it at the point the camera should rotate *around* — roughly the driver's head for first person, or the center of the car for a chase camera.

**2. Attach your camera to it.** For third person, attach a spring arm to the pivot and the camera to the arm, so you still get the arm's collision handling. For first person, attach the camera directly.

**3. Send look input to the pivot** instead of to control rotation. Wire your mouse and stick axes into `AddPitchInput` and `AddYawInput`.

**4. Set the locks.** A chase camera normally wants **Lock Roll** on. Movement is tuned after that.
<!-- split -->
![Camera pivot component on a vehicle with a spring arm and camera attached beneath it](../Assets/Images/_placeholder.png "Pivot first, then spring arm, then camera")
<!-- /side-by-side -->

That is the whole setup. Everything after this is tuning.



## Input Config: Rotation Limits and Axis Locks

<!-- side-by-side:57 -->
**Limit Rotation** enables **Input Rotation Limits**, in degrees, as **X = Roll, Y = Pitch, Z = Yaw**. Each is a symmetrical clamp in both directions, and anything of 180 or above means unlimited on that axis.

The default `900, 65, 135` is unlimited roll, pitch clamped to 65 degrees, and yaw clamped to 135.

Without a pitch limit, players can look straight up or down and lose sight of the vehicle.

A locked axis skips its own limit, since the lock already holds it at zero.

**The world space locks** apply after input, in world space. **Lock Roll** holds the camera's roll at zero no matter what the vehicle is doing, so the horizon stays level. With it off, a cockpit view rolls with the vehicle.

**Relative To Actor** (on by default) uses actor rotation as the reference frame, which makes the component's default rotation a meaningful reset position for `ResetPivotRotation`.
<!-- split -->
![Camera pivot input config showing rotation limits and the roll, pitch and yaw locks](../Assets/Images/_placeholder.png "Lock Roll keeps the horizon level when the vehicle does not")
<!-- /side-by-side -->

Input functions: `AddInput` for all three axes, `AddPitchInput` / `AddYawInput` / `AddRollInput` for one, `ResetPivotRotation` to return to default, and `GetPivotInternalRotation` to read the result.

**Detect Active Camera** (on by default) skips the pivot's work while its camera is inactive. On BeginPlay the pivot searches its children for a camera component, including through a spring arm, so the usual setup needs no wiring.

If more than one camera lives under the same pivot, call `SetTrackedCamera` to say which one counts.

With **Detect Active Camera** on and no camera beneath the pivot, the pivot never moves. That is the usual cause of a pivot that appears to do nothing.



## Movement Modes: None, Lag and Bounce

Location and rotation each have their own mode, set independently.

<!-- side-by-side:50 -->
**Lag (Trailing)**

The pivot smoothly chases where it should be. One setting: **Location Lag Speed** or **Rotation Lag Speed**, in cm/s and deg/s.

Higher is tighter, lower is floatier.

This is the cheaper and more predictable of the two, and is usually all a chase camera needs.
<!-- split -->
**Bounce (G-Force)**

An actual spring simulation. The camera leans under acceleration, dips under braking, and settles after bumps.

**Stiffness** is how hard it corrects. **Damping** is how fast it stops oscillating.

Low stiffness with high damping lets an impact move the camera a long way without it oscillating on the way back.
<!-- /side-by-side -->

### Max Location Offset, Max Rotation Offset and Spring Scale

**Max Location Offset** (cm) and **Max Rotation Offset** (degrees, as X = Roll, Y = Pitch, Z = Yaw) cap how far the pivot can move from its default. They apply to **both** Lag and Bounce.

The defaults are 20 cm and 30 degrees. Low values hold the camera close to its default position and stop a strong spring from pushing it inside or far outside the vehicle. High values let the spring or the lag run to its full travel, so terrain and collisions can throw the camera a long way from its default before it recovers.

**Spring Scale** adjusts the result without re-tuning stiffness and damping. It multiplies the final offset per axis, not the simulation — `1.0` applies the simulated movement fully, `2.0` exaggerates it, `0.5` halves it, `0.0` disables that axis entirely.

An axis at `0.0` does not move at all. For a camera that dips under braking but never sways sideways, set X and Z on **Location Spring Scale** and leave Y at zero.

**Max Linear Velocity** and **Max Angular Velocity** clamp spring speed, which prevents a heavy impact from throwing the camera.



## Example Camera Setups

The same component covers a tight arcade chase camera and a subtle simulation cockpit. The difference is which mode each half uses, and how far the pivot is allowed to move from its default.

| Style | Location | Rotation | Result |
|---|---|---|---|
| **Arcade racer** | `Lag`, fast | `Lag`, fast | Stays close behind the vehicle and returns quickly |
| **Simulation cockpit** | `Bounce`, stiff, small offsets | `Bounce`, small offsets | Small movement, the horizon stays stable |
| **Off-road or heavy vehicle** | `Bounce`, low stiffness, large offsets | `Lag` | Thrown hard by terrain and collisions, snaps back quickly |
| **Cinematic or replay** | `Lag`, slow, large offsets | `Lag`, slow | Drifts well off the vehicle and catches up slowly |

**Location Lag Speed** and **Location Stiffness** set how quickly the camera catches up. 
**Max Location Offset** and **Max Rotation Offset** set how far it can move from its default before the clamp stops it. 
A tight camera is a fast catch-up with a small offset. A large offset lets rough terrain and collisions throw the camera much further before the clamp stops it.

### Arcade Racer Chase Camera

<!-- side-by-side:50 -->
The camera stays close behind the vehicle and returns quickly, which keeps the road ahead visible at speed.

`Lag` on both halves is cheaper than the spring, and it does not overshoot.

The offsets are small, so the camera falls behind only slightly through a corner before returning.

**Lock Roll** keeps the horizon level through landings and wall impacts.
<!-- split -->
- **Location Mode** `Lag`, **Location Lag Speed** `15`
- **Max Location Offset** `60, 60, 30`
- **Rotation Mode** `Lag`, **Rotation Lag Speed** `12`
- **Max Rotation Offset** `5, 10, 25`
- **Lock Roll** on
<!-- /side-by-side -->

A kart or party racer exaggerates the swing into corners. Raise the offsets and lower the lag speeds together.

Raising the offsets alone lets the camera fall further behind but recover at the same speed. Lowering the lag speeds alone leaves it sitting at the offset limit.

### Simulation Cockpit Camera

<!-- side-by-side:50 -->
Cockpit movement is measured in centimeters. Larger offsets move the view enough to make the horizon hard to read.

`Bounce` with a stiff spring and heavy damping recovers quickly rather than floating, and the small **Max Location Offset** caps how far it can travel.

**Location Spring Scale** then trims each axis. Full response fore and aft, half for vertical and lateral.

The locks are off, so the camera rolls with the vehicle.
<!-- split -->
- **Location Mode** `Bounce`, **Location Stiffness** `300`, **Location Damping** `20`
- **Max Location Offset** `5, 5, 5`
- **Location Spring Scale** `1.0, 0.5, 0.5`
- **Rotation Mode** `Bounce`, **Max Rotation Offset** `3, 3, 3`
- **Limit Rotation** on, all locks off
<!-- /side-by-side -->

### Cinematic and Replay Camera

<!-- side-by-side:50 -->
Slow lag speeds with large offsets let the camera drift well off the vehicle before it catches up.

`Lag` rather than `Bounce`, so the drift stays steady instead of reacting to individual bumps.

This normally goes on its own pivot and camera rather than retuning the gameplay one, switched the same way the two perspectives switch below.
<!-- split -->
- **Location Mode** `Lag`, **Location Lag Speed** `3`
- **Max Location Offset** `400, 400, 200`
- **Rotation Mode** `Lag`, **Rotation Lag Speed** `3`
- **Max Rotation Offset** `15, 20, 90`
- **Lock Roll** on
<!-- /side-by-side -->

### Off-Road Vehicle Chase Camera

The two setups below are the actual shipping settings from *[REDACTED]*, a game built on AVS that wants a grounded, heavy feel.

The chase camera is the off-road row of the table. **Location Stiffness** is very low and **Max Location Offset** is raised high. This allows the camera to spring aggressively while driving on rough terrain or from collisions, giving a whiplash effect.

**Location Damping** of `15` against that low stiffness returns the camera quickly, so it stays readable at high speed.

**Rotation** uses `Lag` instead, with **Max Rotation Offset** set high for the same reason. Lag provides us with smoothing while the vehicle rotation changes, rather than bouncing the camera rotation with a spring effect.

The spring arm is a child of the pivot, which keeps the arm's collision handling on top of the pivot's movement. The pivot's own pitch of -10 degrees looks slightly down at the vehicle.

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
Pivot_ThirdPersonCam->SetRelativeRotation(FRotator(-10.0, 0.0, 0.0));

CameraArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraArm"));
CameraArm->SetupAttachment(Pivot_ThirdPersonCam);
CameraArm->TargetArmLength = 420.0f;

ThirdPersonCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("ThirdPersonCamera"));
ThirdPersonCamera->SetupAttachment(CameraArm);
ThirdPersonCamera->SetAutoActivate(false);
```

An arcade racer would keep the structure and change the numbers: `Lag` on both halves, the offsets back down to tens of centimeters, and the lag speeds up.

### Off-Road Vehicle First Person Camera

The camera is inside the cab, so the pivot moves enough to convey the ride without obscuring the view. It is looser on every axis than the cockpit settings above.

**Location** is on `Bounce` with a stiff spring and heavy damping, then **Location Spring Scale** trims each axis separately: mostly forward and back, a little vertical, and very little side to side.

**Rotation** is also on `Bounce`, capped at 15 degrees on every axis.

**Relative To Actor** is off, so look input is measured from the pivot's own default rotation rather than the vehicle's. 
**Limit Rotation** then clamps how far the player can look on each axis.

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

### Switching Between Two Camera Pivots

Each pivot owns one camera, and **Detect Active Camera** means only the active camera's pivot does any work. Switching perspective is then just activating one camera and deactivating the other.

```cpp
void ARTT_Vehicle::UpdateCameraPerspective(const bool bFirstPerson)
{
	FirstPersonCamera->SetActive(bFirstPerson);
	ThirdPersonCamera->SetActive(!bFirstPerson);
}
```

Both pivots are also reset when the controller changes, so a player entering the vehicle starts looking forward rather than wherever the last driver left the camera.

```cpp
void ARTT_Vehicle::NotifyControllerChanged()
{
	Super::NotifyControllerChanged();
	Pivot_FirstPersonCam->ResetPivotRotation();
	Pivot_ThirdPersonCam->ResetPivotRotation();
}
```

Start with small values. The settings in the release screenshots are exaggerated to demonstrate the component, and are stronger than you want while driving.
