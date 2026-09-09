# Recommended Project Settings

These changes dramatically improve physics stability and let wheels reach the speeds they are physically capable of. None of them are strictly required, but the physics wheel mode in particular benefits a lot.

## Min/Max Contact Settings

<!-- side-by-side:41 -->
![Engine - Physics settings with Contact Offset Multiplier 0.001, Min Contact Offset 0.0001 and Max Contact Offset 0.001](../assets/images/general-project-settings-01.png "Contact offsets dropped to Unreal's minimums")
<!-- split -->
When using **Physics Wheels** (not Raycast Wheels), it is strongly recommended to lower the engine's physics contact offset values to the minimum Unreal allows.

By default, Unreal applies a small invisible gap between physics bodies during collision to improve stability. For rolling wheels this has an unwanted side effect: wheels bounce when crossing between separate surfaces, even when those surfaces are perfectly level and aligned. The faster the vehicle moves, the more noticeable the bounce becomes.

Reducing both **Min Contact Offset** and **Max Contact Offset** under _Project Settings → Physics_ significantly reduces or eliminates this behavior.

> This only applies to **Physics Wheel** mode — Raycast Wheels are unaffected by contact offset settings.
>
> The fix was confirmed on UE4. UE5 behavior may differ and hasn't been extensively tested, so verify and adjust as needed.
<!-- /side-by-side -->

## Max Terminal and Angular Velocities

<!-- side-by-side:41 -->
![Engine - Physics settings with Default Terminal Velocity 7600 and Max Angular Velocity 10000](../assets/images/general-project-settings-02.png)
<!-- split -->
Unreal applies limits to how fast physics objects can move and spin. The defaults are too low for realistic vehicles and can stop wheels from reaching the speeds they're capable of.

A wheel with a 35 cm radius spinning at 10000 degrees per second reaches a linear surface speed of about 6105 cm/s (≈ 136 MPH). If your wheels need to push past that, raise the value further.

Set the value high enough that wheels never hit the ceiling under normal operation.
<!-- /side-by-side -->

Update these under _Project Settings → Physics_:

| Setting | Recommended value | Roughly |
|---|---|---|
| Default Terminal Velocity | `7600` cm/s | ≈ 170 MPH |
| Max Angular Velocity | `10000` deg/s | ≈ 136 MPH at a 35 cm wheel radius |

To calculate it yourself:

```text
Linear Speed (cm/s) = (Angular Velocity in deg/s × (π / 180)) × Radius in cm
```

## Physics Tick Settings

Pick one of the two options below. Both keep physics running at a consistent, stable rate, which dramatically reduces physics errors and keeps vehicles stable at low frame rates.

### Option 1: Substepping

<!-- side-by-side:41 -->
![Engine - Physics framerate settings with Substepping enabled, Max Substep Delta Time 0.016667 and Max Substeps 6](../assets/images/general-project-settings-03.png "The 60 FPS physics configuration from the table below")
<!-- split -->
Substepping divides a single game frame into multiple physics ticks, so physics can tick several times per frame to reach its target FPS. This is a good option if you want physics to match the game's frame rate most of the time but still stabilize when it drops.

With substepping, physics is locked to your game's FPS whenever that is above the target. Below the target, substepping gets as close to the target as the resources you allow it will permit.
<!-- /side-by-side -->

| Max Substep Delta Time | Max Substeps | Physics rate | Stable at |
|---|---|---|---|
| `0.016667` | 6 | 60 FPS | 15 / 30 FPS |
| `0.013000` | 6 | 90 FPS | 15 / 30 / 60 FPS |
| `0.009400` | 8 | 120 FPS | 15 / 30 / 60 / 90 FPS |

### Option 2: Async Physics

<!-- side-by-side:48 -->
Async physics fully decouples physics from frame rate. Physics is calculated on a separate thread, sent back to the game thread, then interpolated for visual smoothness. If you need the best possible consistency, choose this.

> Do not use async physics in AVS 1.3 (UE5.0 – 5.1).
>
> Async physics in Unreal 5.3 is broken — stick with substepping.
<!-- split -->
![Tick Physics Async enabled with Async Fixed Time Step Size 0.011111, with an on-screen readout of 91 physics FPS against 60 game FPS](../assets/images/general-project-settings-04.png "Physics running at 90 while the game renders at 60")
<!-- /side-by-side -->

| Async Fixed Time Step Size | Physics rate |
|---|---|
| `0.033333` | 30 FPS |
| `0.016667` | 60 FPS |
| `0.011111` | 90 FPS |
| `0.008333` | 120 FPS |

That should be it. AVS takes care of the other relevant settings internally where it can.
