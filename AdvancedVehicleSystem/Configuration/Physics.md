# Physics

Vehicle level physics — center of mass, body limits, and moving a vehicle safely. Wheel and suspension settings live on the wheel components and are covered on the [Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md) page.

Before tuning anything here, apply the [recommended project settings](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Project_Settings.md). Most "my vehicle is unstable" problems are solved there, not on this page.



## Center of Mass Placement

Center of mass placement affects handling more than any other single setting, and it is worth checking first when a vehicle handles badly for no obvious reason.

<!-- side-by-side:57 -->
**1. Add a [Center of Mass component](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Center_Of_Mass.md)** to the vehicle. It is a marker with no settings — you position it and AVS uses its location.

**2. Put it slightly above wheel height**, roughly where the mass actually sits. This gives a stable but responsive feel and is a good starting point for most vehicles.

**3. Turn on Visualize Center of Mass** under **Advanced Vehicle System → Debug** and check where it actually ended up. This draws the real physics COM after every offset, and it is the only reliable confirmation.

**4. Adjust front to back.** Moving mass rearward adds rear grip and encourages understeer; forward does the opposite.
<!-- split -->
![Vehicle in the viewport with the center of mass visualizer enabled, showing the marker below the chassis](../Assets/Images/_placeholder.png "Visualize Center of Mass draws the real COM, after all offsets")
<!-- /side-by-side -->

Height is a trade, not a value to minimise:

| Placement | Result |
|---|---|
| Too high | Rolls over in corners |
| Slightly above wheel height | Stable but responsive — the usual target |
| Below the wheels | Resists flipping, but leans the wrong way through corners |

A vehicle that leans outward through a corner looks wrong even when it drives correctly, so placing the center of mass as low as possible is not automatically better.

### Setting Center of Mass from Code

Three sources, in order of precedence:

1. `SetExactCenterOfMass` — explicit override relative to the vehicle mesh pivot. Beats everything.
2. The Center of Mass component's location.
3. The vehicle mesh pivot, when neither of the above exists.

`SetCenterOfMassOffset` adds a further offset on top of whichever base applies. It **sets rather than accumulates**, so calling it repeatedly will not drift — useful for shifting mass with cargo load.

`GetExactCenterOfMass` returns the real value with a validity flag. It is invalid until a physics body exists, so do not call it during construction.



## Vehicle Physics Settings

| Setting | Default | What it does |
|---|---|---|
| **Start With Physics** | `true` | Vehicle simulates on spawn. Exposed on spawn. |
| **Physics Material** | none | Overrides the vehicle mesh's physical material, so you do not have to edit the mesh asset. |
| **Vehicle Max Angular Velocity** | `700` deg/s | Caps how fast the body can spin. `0` disables the limit. |
| **Wheels Push Physics** | `true` | Whether wheels physically push other simulating objects. |
| **Disable Skeletal Collisions** | `true` | See [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md). |

Two of these need explanation:

**Wheels Push Physics** allows a vehicle to push loose props. It also causes a see-saw effect when a vehicle drives onto another physics object, because the wheels push the object and the object pushes back. Turn it off if that happens.

**Vehicle Max Angular Velocity** limits how fast the body spins after a heavy crash. It is separate from the engine's global Max Angular Velocity, which applies to wheels. If a vehicle will not reach top speed, the engine setting is the one to check.

### Advanced Physics Settings

**Rest Velocity Threshold** (`25` cm/s) decides when a vehicle counts as stopped, for both passive mode and network rest state. Raise it and vehicles settle sooner, which helps performance but can cut off slow creeping. Lower it if vehicles sleep while they should still be rolling.

**Upside Down Angle Threshold** (`90` deg) sets when the vehicle counts as flipped — `0` is upright, `180` fully inverted, and the default counts a vehicle on its side. `GetIsUpsideDown` reads it, and **OnVehicleFlipped** fires when it changes, which is where a "press R to recover" prompt belongs.



## Teleporting and Repositioning a Vehicle

Do not set the actor transform directly on a simulating vehicle. The wheels do not follow, and the physics state ends up incorrect.

<!-- side-by-side:50 -->
**Teleporting**

`TeleportVehicle(Location, Rotation, KeepRelativeVelocity)` moves the vehicle and its wheels together.

`KeepRelativeVelocity` decides whether it arrives moving or stopped — keep it for a portal, drop it for a respawn.
<!-- split -->
**Aligning to a marker**

`SetVehiclePositionWithVirtualPivot(LocalPivot, WorldTransform, bTeleport)` positions the vehicle so a chosen local pivot lands on a target world transform.

Use it when your spawn marker represents something other than the vehicle origin — a wheel contact point, or a trailer coupler.
<!-- /side-by-side -->

`SetPhysics(bool)` toggles simulation entirely, which is what you want before attaching a vehicle to something else or handing it to a sequencer.



## Drag and Damping

**Base Linear Drag** is the fallback linear damping, and **Dynamic Air Drag** enables additional slip-based damping at higher speeds. Both are runtime values rather than editor settings, so set them from Blueprint or C++ when drag should change with vehicle state — a spoiler deploying, or a load being dropped.

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
