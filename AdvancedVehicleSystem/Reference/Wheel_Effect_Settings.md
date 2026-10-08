# Wheel Effect Settings

Every setting on the built-in wheel effects and on the output block they share. For how to set effects up, see [Wheel Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Overview.md).



## Vehicle Effect Slots

On the vehicle, under **Advanced Vehicle System → Wheel Effects**.

| Setting | Description |
|---|---|
| **Global Wheel Effects** | Effects that run on every surface. |
| **Effects For Default Surface** | Effects for any surface without its own entry in **Surface Effects**. |
| **Surface Effects** | Effects for specific physical surfaces. A `Default` key here is never read; use **Effects For Default Surface**. |

Each wheel can replace any of these. See the **Wheel Effects** section of [Wheel Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Settings.md).



## Roll

Runs while the wheel has contact. Intensity follows how fast the wheel is moving.

| Setting | Default | Description |
|---|---|---|
| **Min Speed** | `100` cm/s | Where the effect begins. |
| **Full Speed** | `2000` cm/s | Where the effect reaches full intensity. It stays at full above this. |

Uses the shared output block.



## Skid / Slip

Runs while the wheel has contact. Intensity follows the selected **Slip Source**.

| Setting | Default | Description |
|---|---|---|
| **Slip Source** | `Skid Speed (Classic)` | What the effect responds to. See below. |
| **Min Skid Speed** | `500` cm/s | Classic source. Where the effect begins. |
| **Full Skid Speed** | `1500` cm/s | Classic source. Where it reaches full intensity. |
| **Min Slip** | `0.5` | Calculated sources. Where the effect begins. |
| **Full Slip** | `1.5` | Calculated sources. Where it reaches full intensity. |
| **Response Speed** | `5.0` | How quickly intensity follows changes. Higher reacts faster; lower smooths brief changes. |
| **Min Wheel Speed** | `100` cm/s | Below this wheel speed, nothing plays. |

| Slip Source | Responds to |
|---|---|
| `Skid Speed (Classic)` | The wheel's sideways speed, or its full speed while the wheel is locked. Works in both wheel modes. |
| `X Slip` | Slip along the wheel: wheelspin and lockup. Raycast wheels only. |
| `Y Slip` | Slip across the wheel: sliding. Raycast wheels only. |
| `Combined Slip` | Total slip. Raycast wheels only. |

The settings for the source you are not using are hidden. Uses the shared output block.



## Bump / Impact

Runs while the wheel has contact. Fires once for each impact.

| Setting | Default | Description |
|---|---|---|
| **Min Strength** | `200` cm/s | Impact speed into the surface where the effect begins. |
| **Full Strength** | `1000` cm/s | Impact speed that produces full intensity. |
| **Cooldown** | `0.2` s | Minimum time between impacts. Resets when the wheel leaves the ground. |

Uses the shared output block.



## Brake Squeal

Runs on braking wheels, on any surface. Audio only, with its own output settings.

| Setting | Default | Description |
|---|---|---|
| **Brake Power Range** | `10000, 150000` W | Brake power where squeal can begin, and full workload. |
| **Wheel Speed Range** | `100, 1000` cm/s | Wheel speed where squeal begins, and where it reaches full. |
| **Squeal Propensity** | `1` | Brake condition, from `0` to `1`. Change it at runtime to simulate wear. Higher values squeal at lower brake power, and louder. `0` disables squeal. |
| **Sound** | none | The squeal sound. |
| **Attenuation** | none | Sound attenuation. |
| **Intensity Param** | none | Optional audio parameter, `0` to `1`, from the square of **Squeal Propensity**. Suited to volume. |
| **Brake Power Param** | none | Optional audio parameter, `0` to `1`, across **Brake Power Range**. |
| **Wheel Speed Param** | none | Optional audio parameter, `0` to `1`, across **Wheel Speed Range**. Suited to pitch. |
| **Fade In Duration** | `0.1` s | Time for the squeal to reach full volume. |
| **Fade Out Duration** | `0.1` s | Time for the squeal to fade out. |
| **Random Start Time Max** | `3` s | Random start position in the sound, so wheels do not play in sync. |



## Shared Output Block

Roll, Skid / Slip and Bump / Impact share these settings.

| Setting | Default | Description |
|---|---|---|
| **Attach To Wheel** | off | On, the effect follows the wheel. Off, it stays where it spawned. |
| **Use Contact Point** | on | Spawns at the contact point. Off, or when there is no contact, spawns at the wheel component. |
| **Location Offset** | `0, 0, 0` cm | Offset in the wheel's own directions. Corrected for **Invert Torque** wheels. |
| **Rotation Offset** | `0, 0, 0` | Rotation relative to the wheel's forward. Corrected for **Invert Torque** wheels. |
| **Parameter Intensity Range** | `0, 1` | Maps the effect's `0` to `1` intensity into the range your audio and particle parameters expect. |
| **Sound** | none | The sound to play. |
| **Attenuation** | none | Sound attenuation. |
| **Audio Intensity Param** | none | Optional audio parameter that receives the mapped intensity. |
| **Audio Fade Out Duration** | `0.1` s | Fade time for continuous audio before it is released. |
| **Random Start Time Max** | `0` s | Random start position in the sound. Set it to the loop's length so wheels do not play in sync. |
| **Niagara System** | none | The Niagara system to spawn. |
| **Niagara Intensity Param** | none | Optional Niagara float parameter that receives the mapped intensity. |
| **Cascade System** | none | The Cascade system to spawn. |
| **Cascade Intensity Param** | none | Optional Cascade float parameter that receives the mapped intensity. |



## Effect Functions

Available on any wheel effect, including your own:

| Function | Description |
|---|---|
| `StartOutput(Output, Intensity, bContinuous)` | Spawns and starts audio and particles. **bContinuous** is true for looping effects. |
| `UpdateOutput(Output, Intensity)` | Updates the intensity of a running output. |
| `StopOutput()` | Fades audio out and deactivates particles. |
| `IsOutputActive()` | Whether output is running. |
| `GetOutputTransform(Output)` | The world transform the output's placement settings resolve to. |
| `GetContactStartedThisFrame()` | True on the frame the wheel regains contact. |
| `GetContactImpactSpeed()` | The strongest current or completed impact this frame, in cm/s. |

On the vehicle, `ClearAllWheelEffects()` stops every wheel's effects. They are rebuilt on the next tick.
