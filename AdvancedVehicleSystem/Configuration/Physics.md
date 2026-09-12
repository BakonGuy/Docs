# Physics

Vehicle level physics — center of mass, body limits, and moving a vehicle safely. Wheel and suspension settings live on the wheel components and are covered on the [Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md) page.

Before tuning anything here, apply the [recommended project settings](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Project_Settings.md). Most "my vehicle is unstable" problems are solved there, not on this page.



## Basic Understanding

Center of mass placement affects handling more than any other single setting, and it is worth checking when a vehicle handles badly for no obvious reason.

Everything else on this page falls into two groups. **Body limits** — angular velocity, rest threshold, flip threshold — control what the physics body is allowed to do. **Movement functions** — teleporting, repositioning, toggling simulation — are how you move a vehicle without breaking its physics state.



## Setting the Center of Mass

<!-- side-by-side:57 -->
**1. Add a [Center of Mass component](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Center_Of_Mass.md)** to the vehicle. It is a marker with no settings — you position it and AVS uses its location.

**2. Put it slightly above wheel height**, roughly where the mass actually sits. This gives a stable but responsive feel and is a good starting point for most vehicles.

**3. Turn on Visualize Center of Mass** under **Advanced Vehicle System → Debug** and check where it actually ended up. This draws the real physics COM after every offset, and it is the only reliable confirmation.

**4. Adjust front to back.** Moving mass rearward adds rear grip and encourages understeer; forward does the opposite.
<!-- split -->
![Vehicle in the viewport with the center of mass visualizer enabled, showing the marker below the chassis](../Assets/Images/_placeholder.png "Visualize Center of Mass draws the real COM, after all offsets")
<!-- /side-by-side -->



## Center of Mass Height

Height is a trade, not a value to minimise.

Too high and the vehicle rolls over in corners. Slightly above wheel height is stable but responsive, which is the usual target. Below the wheels it resists flipping, but leans the wrong way through a corner.

A vehicle that leans outward through a corner looks wrong even when it drives correctly, so placing the center of mass as low as possible is not automatically better.



## Center of Mass Precedence

Three sources, in order of precedence:

1. `SetExactCenterOfMass` — explicit override relative to the vehicle mesh pivot. Beats everything.
2. The Center of Mass component's location.
3. The vehicle mesh pivot, when neither of the above exists.

`GetExactCenterOfMass` returns the real value with a validity flag. It is invalid until a physics body exists, so do not call it during construction.



## SetCenterOfMassOffset

`SetCenterOfMassOffset` adds a further offset on top of whichever base applies.

It **sets rather than accumulates**, so calling it repeatedly will not drift. This is what you want for shifting mass with cargo load.



## Vehicle Mass

Mass is set on the vehicle mesh component, in Unreal's own physics settings, rather than by AVS. AVS does not modify it — it applies forces to the body you give it.

Use realistic figures. A vehicle under `1000` kg is very light, and is the usual cause of a car that feels skittish or gets shoved around by other physics objects.

A compact car starts around `1100` kg, a midsize sedan around `1500`, and a pickup or van around `2000`.

Set mass before tuning tire friction. Friction values that feel right on a 900 kg car will not suit the same car at 1500 kg, so tuning grip first means doing it twice.

A light vehicle is a legitimate choice for arcade handling. It is only a problem when it is unintentional.



## Vehicle Physics Settings

| Setting | Default | What it does |
|---|---|---|
| **Start With Physics** | `true` | Vehicle simulates on spawn. Exposed on spawn. |
| **Physics Material** | none | Overrides the vehicle mesh's physical material, so you do not have to edit the mesh asset. |
| **Vehicle Max Angular Velocity** | `700` deg/s | Caps how fast the body can spin. `0` disables the limit. |
| **Wheels Push Physics** | `true` | Whether wheels physically push other simulating objects. |
| **Disable Skeletal Collisions** | `true` | See [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md). |



## Wheels Push Physics

**Wheels Push Physics** allows a vehicle to push loose props.

It also causes a see-saw effect when a vehicle drives onto another physics object, because the wheels push the object and the object pushes back. Turn it off if that happens.



## Vehicle Max Angular Velocity

**Vehicle Max Angular Velocity** limits how fast the body spins after a heavy crash.

It is separate from the engine's global Max Angular Velocity, which applies to wheels.

> If a vehicle will not reach top speed, the engine's global setting is the one to check, not this one.



## Rest Velocity Threshold

**Rest Velocity Threshold** (`25` cm/s) decides when a vehicle counts as stopped, for both passive mode and network rest state.

Raise it and vehicles settle sooner, which helps performance but can cut off slow creeping.

Lower it if vehicles sleep while they should still be rolling.



## Upside Down Angle Threshold

**Upside Down Angle Threshold** (`90` deg) sets when the vehicle counts as flipped. `0` is upright, `180` fully inverted, and the default counts a vehicle on its side.

`GetIsUpsideDown` reads it, and **OnVehicleFlipped** fires when it changes, with `bIsNowUpsideDown` telling you which way.

AVS detects the state and tells you about it. What happens next is yours to write — there is no built-in righting function. **OnVehicleFlipped** is where a "press R to recover" prompt belongs.



## Teleporting a Vehicle

> Do not set the actor transform directly on a simulating vehicle. The wheels do not follow, and the physics state ends up incorrect.

`TeleportVehicle(Location, Rotation, KeepRelativeVelocity)` moves the vehicle and its wheels together.

`KeepRelativeVelocity` decides whether it arrives moving or stopped — keep it for a portal, drop it for a respawn.



## Aligning a Vehicle to a Marker

`SetVehiclePositionWithVirtualPivot(LocalPivot, WorldTransform, bTeleport)` positions the vehicle so a chosen local pivot lands on a target world transform.

Use it when your spawn marker represents something other than the vehicle origin — a wheel contact point, or a trailer coupler.



## Enabling and Disabling Physics

`SetPhysics(bool)` toggles simulation entirely.

This is what you want before attaching a vehicle to something else or handing it to a sequencer.



## Drag and Damping

**Base Linear Drag** is the fallback linear damping, and **Dynamic Air Drag** enables additional slip-based damping at higher speeds.

Both are runtime values rather than editor settings, so set them from Blueprint or C++ when drag should change with vehicle state — a spoiler deploying, or a load being dropped.

> Damping was reworked in 1.5. Suspension damping now uses the spring's actual compression and extension instead of estimating from movement at the contact point. If you had instability around 140 MPH on an older version, that was the old approximation leaking velocity.



## Reading Physics State

| Function | Returns |
|---|---|
| `GetAirSpeed` | Actual speed, in your chosen speed unit |
| `GetAcceleration` | Change in speed per second |
| `GetSlip` | General lateral slip for the whole vehicle |
| `AnyWheelContact` | Whether any wheel is touching a surface |
| `GetPhysicsTickDelta` | Current Chaos substep delta |

`GetSlip` is an approximate whole-vehicle value. For accurate results, such as traction control or per-corner effects, read slip from the wheels instead.
