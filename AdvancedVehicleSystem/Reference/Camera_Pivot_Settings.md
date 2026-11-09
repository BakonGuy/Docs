# Camera Pivot Settings

Every setting and function on the `AVS_CameraPivot` component, grouped the way the details panel groups them. For how to set a pivot up, see [Camera Pivot](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Camera_Pivot.md).

Rotation vectors are X = Roll, Y = Pitch, Z = Yaw.



## Input Config

| Setting | Default | Description |
|---|---|---|
| **Relative To Actor** | on | On, look input is measured from the vehicle's rotation, and `ResetPivotRotation` applies the pivot's own rotation. Off, look input is measured from the pivot's own rotation. |
| **Limit Rotation** | off | Clamps look input to **Input Rotation Limits**. |
| **Input Rotation Limits** | `900, 65, 135` | Degrees each way on each axis. `180` or more leaves that axis unlimited. |
| **Lock Roll** | off | Holds the pivot's roll at `0` in world space. |
| **Lock Pitch** | off | Holds the pivot's pitch at `0` in world space. |
| **Lock Yaw** | off | Holds the pivot's yaw at `0` in world space. |
| **Detect Active Camera** | on | The pivot only updates while its camera is active. With no camera found, it never updates. Advanced setting. |

A locked axis skips its rotation limit.

With **Relative To Actor** on, the pivot's own rotation is not applied until `ResetPivotRotation` is called. With it off, `ResetPivotRotation` applies the pivot's rotation a second time on top of the reference frame.



## Movement

| Setting | Default | Description |
|---|---|---|
| **Location Mode** | `None (Static Location)` | `None (Static Location)`, `Lag (Trailing)` or `Bounce (G-Force)`. |
| **Rotation Mode** | `None (Static Location)` | The same three modes, for rotation. |

`Lag` smoothly follows the target. It never overshoots. `Bounce` is a spring pushed by the vehicle's acceleration.



## Location Mode Config

| Setting | Default | Description |
|---|---|---|
| **Location Lag Speed** | `10` | `Lag` only. How fast the pivot catches up. Higher is faster. |
| **Max Location Offset** | `20, 20, 20` cm | How far the pivot can move from its default position, in both `Lag` and `Bounce`. |
| **Location Stiffness** | `100` | `Bounce` only. How hard the spring pulls back. |
| **Location Damping** | `10` | `Bounce` only. How quickly the spring stops oscillating. |
| **Location Spring Scale** | `1, 1, 1` | `Bounce` only. Multiplies the spring's offset on each of the pivot's axes (X forward, Y right, Z up). `0` removes movement on that axis. |
| **Max Linear Velocity** | `700` cm/s | `Bounce` only. How fast the spring can move. |



## Rotation Mode Config

| Setting | Default | Description |
|---|---|---|
| **Rotation Lag Speed** | `10` | `Lag` only. How fast the rotation catches up. Higher is faster. |
| **Max Rotation Offset** | `30, 30, 30` degrees | How far the rotation can move from its target, in both `Lag` and `Bounce`. |
| **Rotation Stiffness** | `100` | `Bounce` only. How hard the spring pulls back. |
| **Rotation Damping** | `10` | `Bounce` only. How quickly the spring stops oscillating. |
| **Rotation Spring Scale** | `1, 1, 1` | Shown for `Bounce`, but currently has no effect. |
| **Max Angular Velocity** | `700` deg/s | `Bounce` only. How fast the spring can rotate. |



## Camera Pivot Functions

| Function | Description |
|---|---|
| `AddInput(Roll, Pitch, Yaw)` | Adds look input on all three axes. |
| `AddPitchInput(Pitch)`, `AddYawInput(Yaw)`, `AddRollInput(Roll)` | Adds look input on one axis. |
| `ResetPivotRotation()` | Returns look input to the pivot's default rotation. |
| `GetPivotInternalRotation()` | The current look input rotation. |
| `SetTrackedCamera(Camera)` | Sets the camera **Detect Active Camera** watches. Needed when more than one camera is under the pivot. |

The pivot finds its camera on BeginPlay by searching all its children, including through a spring arm. With more than one, the last one found is used.
