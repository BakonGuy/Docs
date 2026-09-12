# Camera Pivot

`AVS_CameraPivot` is the camera rig you would otherwise rebuild on every project — look input, what happens when the vehicle flips, and movement like G-force lean or camera lag.



## Basic Understanding

The pivot is a scene component that your camera attaches to. It does three jobs, and each one is configured separately.

**Input.** The pivot holds its own look rotation, fed by `AddPitchInput` and `AddYawInput` instead of control rotation. It can clamp that rotation per axis, and it can lock an axis in world space so the camera keeps a level horizon while the vehicle rolls.

**Movement.** Location and rotation each run in one of three modes — `None`, `Lag` or `Bounce` — which is what produces camera lag or G-force lean.

**Timing.** The pivot ticks in `PostPhysics`, so it always reads the vehicle's final transform for the frame.



## Camera Pivot vs Spring Arm

For a fixed chase camera, a spring arm is usually enough. The pivot is for cameras that need to react to the vehicle.

Camera rigs built from control rotation and spring arms often stutter, because they update at the wrong point relative to the vehicle's physics. The pivot ticks after physics instead. It also keeps look input separate from control rotation, so it doesn't rely on Unreal's rotation handling.

The two work together. A third person setup normally uses both, with the spring arm as a child of the pivot.



## Step 1: Add the Pivot to your Vehicle

<!-- side-by-side:57 -->
Add an `AVS_CameraPivot` component to your vehicle.

Position it at the point the camera should rotate *around* — roughly the driver's head for a first person view, or the center of the vehicle for a chase camera.

The pivot's own transform is its default position. Everything the movement modes do is measured as an offset from it, and `ResetPivotRotation` returns to it.
<!-- split -->
![Add Component menu filtered to "camera", showing AVS Camera Pivot in the list](../Assets/Images/_placeholder.png "Add Component → AVS Camera Pivot")
<!-- /side-by-side -->



## Step 2: Attach your Camera

<!-- side-by-side:57 -->
For a **third person** camera, attach a spring arm to the pivot and the camera to the arm. You keep the arm's collision handling, and the pivot moves the whole assembly.

For a **first person** camera, attach the camera directly to the pivot.

The pivot finds its camera on BeginPlay by searching its children, including through a spring arm, so neither case needs wiring.
<!-- split -->
![Component hierarchy with a spring arm parented under the camera pivot and a camera under the arm](../Assets/Images/_placeholder.png "Pivot first, then spring arm, then camera")
<!-- /side-by-side -->



## Step 3: Send Look Input to the Pivot

Wire your mouse and stick axes into the pivot's input functions rather than into control rotation.

```cpp
// Look input goes to the pivot, not to AddControllerYawInput.
Pivot_ThirdPersonCam->AddYawInput(LookAxis.X);
Pivot_ThirdPersonCam->AddPitchInput(LookAxis.Y);
```

The same functions exist in Blueprint on the pivot component.



## Step 4: Set the Axis Locks

A chase camera normally wants **Lock Roll** on, so the horizon stays level when the vehicle rolls.

That is the whole setup. Everything after this is tuning.



## Input Rotation Limits

<!-- side-by-side:57 -->
**Limit Rotation** enables **Input Rotation Limits**, in degrees, as **X = Roll, Y = Pitch, Z = Yaw**.

Each is a symmetrical clamp in both directions. Anything of 180 or above means unlimited on that axis.

The default `900, 65, 135` is unlimited roll, pitch clamped to 65 degrees, and yaw clamped to 135.

Without a pitch limit, players can look straight up or down and lose sight of the vehicle.
<!-- split -->
![Camera pivot input config showing Limit Rotation enabled and the Input Rotation Limits vector](../Assets/Images/_placeholder.png "Limits are X = Roll, Y = Pitch, Z = Yaw")
<!-- /side-by-side -->

A locked axis skips its own limit, since the lock already holds it at zero.



## World Space Locks: Lock Roll, Lock Pitch and Lock Yaw

The locks apply after input, in world space.

**Lock Roll** holds the camera's roll at zero no matter what the vehicle is doing, so the horizon stays level. This is the usual choice for a chase camera.

With the locks off, the camera rolls and pitches with the vehicle, which is what a cockpit view wants.



## Relative To Actor

**Relative To Actor** (on by default) uses the vehicle's actor rotation as the reference frame for look input.

With it off, the pivot's own default rotation is the reference frame instead. A pivot that is already angled — a chase camera pitched down at the vehicle, for example — then measures input from that angle rather than from the vehicle's.



## Detect Active Camera

**Detect Active Camera** (on by default) skips the pivot's work while its camera is inactive. A vehicle with a first person and a third person pivot only pays for the one in use.

The pivot searches its children for a camera component on BeginPlay. If more than one camera lives under the same pivot, call `SetTrackedCamera` to say which one counts.

> With **Detect Active Camera** on and no camera beneath the pivot, the pivot never moves. That is the usual cause of a pivot that appears to do nothing.



## Camera Pivot Input Functions

| Function | Does |
|---|---|
| `AddInput(Roll, Pitch, Yaw)` | Adds look input on all three axes |
| `AddPitchInput` / `AddYawInput` / `AddRollInput` | Adds look input on one axis |
| `ResetPivotRotation` | Returns the pivot to its default rotation |
| `GetPivotInternalRotation` | Reads the current input rotation |
| `SetTrackedCamera` | Sets which camera **Detect Active Camera** watches |



## Movement Modes: None, Lag and Bounce

Location and rotation each have their own mode, set independently. A camera can lag its position while holding its rotation fixed, or the reverse.

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

`None` holds the pivot at its default. A camera that should not move at all uses `None` for both.



## Max Location Offset and Max Rotation Offset

<!-- side-by-side:57 -->
**Max Location Offset** (cm) and **Max Rotation Offset** (degrees, as X = Roll, Y = Pitch, Z = Yaw) cap how far the pivot can move from its default. They apply to **both** Lag and Bounce.

The defaults are 20 cm and 30 degrees.

Low values hold the camera close to its default position and stop a strong spring from pushing it inside or far outside the vehicle.

High values let the spring or the lag run to its full travel, so terrain and collisions can throw the camera a long way from its default before it recovers.
<!-- split -->
![Camera pivot movement config showing the location mode, lag speed and max location offset](../Assets/Images/_placeholder.png "The offsets cap every mode, not just Bounce")
<!-- /side-by-side -->



## Location Spring Scale and Rotation Spring Scale

**Spring Scale** adjusts the result without re-tuning stiffness and damping. It multiplies the final offset per axis, not the simulation.

- `1.0` applies the simulated movement fully
- `2.0` exaggerates it
- `0.5` halves it
- `0.0` disables that axis entirely

For a camera that dips under braking but never sways sideways, set X and Z on **Location Spring Scale** and leave Y at zero.

Both scales are Bounce only.



## Max Linear Velocity and Max Angular Velocity

**Max Linear Velocity** (cm/s) and **Max Angular Velocity** (deg/s) clamp how fast the spring itself can move, which prevents a heavy impact from throwing the camera.

Both default to `700`, and both are Bounce only.



## Choosing Settings for your Game

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



## Example: Arcade Racer Chase Camera

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



## Example: Simulation Cockpit Camera

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



## Example: Cinematic and Replay Camera

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



## Example: Off-Road Vehicle Chase Camera

These settings come from a game with a grounded, heavy feel, where rough terrain and collisions are constant.

**Location Stiffness** is very low and **Max Location Offset** is raised high. This allows the camera to spring aggressively while driving on rough terrain or from collisions, giving a whiplash effect.

**Location Damping** of `15` against that low stiffness returns the camera quickly, so it stays readable at high speed.

**Rotation** uses `Lag` instead, with **Max Rotation Offset** set high for the same reason. Lag provides us with smoothing while the vehicle rotation changes, rather than bouncing the camera rotation with a spring effect.

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

The spring arm is a child of the pivot, which keeps the arm's collision handling on top of the pivot's movement. The pivot's own pitch of -10 degrees looks slightly down at the vehicle.

An arcade racer would keep the structure and change the numbers: `Lag` on both halves, the offsets back down to tens of centimeters, and the lag speeds up.



## Example: Off-Road Vehicle First Person Camera

The camera is inside the cab, so the pivot moves enough to convey the ride without obscuring the view. It is looser on every axis than the cockpit settings above.

**Location** is on `Bounce` with a stiff spring and heavy damping, then **Location Spring Scale** trims each axis separately: mostly forward and back, a little vertical, and very little side to side.

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

Both pivots attach to `VehicleMesh` rather than the actor root, and both cameras start deactivated.



## Switching Between Two Camera Pivots

Each pivot owns one camera, and **Detect Active Camera** means only the active camera's pivot does any work. Switching perspective is then just activating one camera and deactivating the other.

```cpp
void AMyVehicle::UpdateCameraPerspective(const bool bFirstPerson)
{
	FirstPersonCamera->SetActive(bFirstPerson);
	ThirdPersonCamera->SetActive(!bFirstPerson);
}
```

Both pivots are also reset when the controller changes, so a player entering the vehicle starts looking forward rather than wherever the last driver left the camera.

```cpp
void AMyVehicle::NotifyControllerChanged()
{
	Super::NotifyControllerChanged();
	Pivot_FirstPersonCam->ResetPivotRotation();
	Pivot_ThirdPersonCam->ResetPivotRotation();
}
```
