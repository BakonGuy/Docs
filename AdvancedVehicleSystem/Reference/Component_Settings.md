# Component Settings

Every setting and function on the smaller AVS components. Wheels and the camera pivot have their own pages: [Wheel Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Settings.md) and [Camera Pivot Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Camera_Pivot_Settings.md).



## AVS_Hitch

See [Hitch / Trailers](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Hitch_Trailers.md).

| Setting | Default | Description |
|---|---|---|
| **Hitch Type** | `Tow Hitch` | `Tow Hitch` is the ball on the towing vehicle. `Trailer Hitch` is the coupler on the trailer. |
| **Connect Types** | `Standard` | Two hitches can connect only if they share at least one entry. |
| **X Rotation Limit** | `45` degrees | How far the trailer can roll at the hitch. `180` leaves it free. Only the tow hitch's value is used. |
| **Y Rotation Limit** | `70` degrees | How far the trailer can pitch at the hitch. |
| **Z Rotation Limit** | `120` degrees | How far the trailer can turn at the hitch. |

| Function | Description |
|---|---|
| `Hitch(To, AnimationLength)` | Connects to another hitch. The trailer is pulled into place over **AnimationLength** seconds (default `2`), then constrained. Owning client or server only. |
| `HitchToOverlapped(AnimationLength)` | Tow hitch only. Hitches to the first unhitched trailer hitch whose collision shape overlaps this hitch's. |
| `Unhitch()` | Releases the connection. |
| `GetIsHitched()` | Whether the hitch is connected. |
| `GetHitchedTo()` | The hitch on the other end. |
| `GetHitchedVehicle()`, `GetHitchedTrailer()` | The towing vehicle and the trailer. |

| Event | Fires |
|---|---|
| **Hitched** | When a connection is made, with the tow hitch and the trailer hitch. |
| **Unhitched** | When the connection is released. |
| **Hitched To Destroyed** | When the hitch on the other end is destroyed. The hitch then unhitches itself. |



## AVS_LightController

See [Lights](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Lights.md).

| Setting | Default | Description |
|---|---|---|
| **Relations** | empty | Rules of the form "while this light group is active, set this intensity". When several groups are active, the last matching rule wins. |

Each relation has **When Group Active** (the light group name) and **Set Intensity**.

| Function | Description |
|---|---|
| `Illuminate(Intensity)` | Sets the intensity of every light under the controller. Replaced by the next `UpdateLights`. |
| `GetIntensity()` | The current intensity. |
| `HasActiveLights()` | Whether the intensity is above `0`. |

Light groups are set on the vehicle: `SetLightsActive`, `GetLightsActive`, `ToggleLightsActive` and `UpdateLights`. See [Vehicle Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Vehicle_Settings.md).



## AVS_EngineAudio

See [Engine Audio](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Engine_Audio.md).

| Setting | Default | Description |
|---|---|---|
| **Engine Attenuation** | `Engine_Att` | Attenuation for both sounds. |
| **Engine Start Sound** | none | Played when the engine starts. Fades out over the last three quarters of its length. |
| **Engine Sound** | none | The engine loop. Receives the **RPM** and **Throttle** parameters. |
| **High RPM** | `7000` | The vehicle RPM sent to the sound as an **RPM** of `8000`. |

| Function | Description |
|---|---|
| `StopSound()` | Stops the engine loop. |
| `SetEngineRunning(Running)` | Currently has no effect. |



## AVS_ExhaustController

See [Exhaust](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Exhaust.md).

The controller has no settings. It drives every Cascade particle system component beneath it:

| Parameter | Value |
|---|---|
| **SmokeVelocity** | Vector. X goes from `40` to `150` as RPM rises from `0` to `10000`. |
| **SmokeSize** | Vector. X goes from `30` to `80` as throttle goes from `0` to `1`. |

The particles are active while the engine is running and the vehicle is slower than `40` in its **Speed Units**.



## AVS_CenterOfMass

See [Center of Mass](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Center_Of_Mass.md).

The component has no settings. Its location sets the vehicle's center of mass when it is attached directly to `VehicleMesh`.
