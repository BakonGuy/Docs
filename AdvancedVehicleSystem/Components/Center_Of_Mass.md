# Center of Mass

`AVS_CenterOfMass` is a marker component with no settings. You add it, drag it where the mass should sit, and AVS uses its location.



## Basic Understanding

The component is a position and nothing else. It exists so you can place the center of mass visually instead of entering coordinates.

It is not the only thing that can set the center of mass — a code override beats it, and an additional offset can be applied on top. The precedence is listed at the bottom of this page.



## Adding a Center of Mass Component

<!-- side-by-side:57 -->
**1. Add the component** to your vehicle.

**2. Position it slightly above wheel height**, roughly where the vehicle's mass actually is. Low in the chassis, between the axles, is a good starting point for a car.

**3. Turn on Visualize Center of Mass** under **Advanced Vehicle System → Debug**, and confirm the drawn COM is where you intended. This draws the *actual* physics center of mass after every offset has been applied — it is the only reliable check.

**4. Drive it, then adjust.** Front to back placement changes the balance: rearward adds rear grip and understeer, forward does the opposite.
<!-- split -->
![Center of mass component positioned inside a vehicle chassis with the visualizer enabled](../Assets/Images/_placeholder.png "Position the marker, then confirm with the visualizer")
<!-- /side-by-side -->



## Center of Mass Height

Lower is not automatically better:

| Placement | Result |
|---|---|
| Too high | Rolls over in corners |
| Slightly above wheel height | Stable but responsive — the usual target |
| Below the wheels | Resists flipping, but leans the wrong way through corners |

A vehicle leaning outward through a corner looks wrong even when it drives correctly. If you need a vehicle that cannot flip, keep the center of mass in a normal position and adjust suspension and grip instead.



## Center of Mass Precedence

Three things can set the center of mass:

1. `SetExactCenterOfMass` — explicit override, relative to the vehicle mesh pivot. Beats the component.
2. **This component's location.**
3. The vehicle mesh pivot, when neither exists.

`GetExactCenterOfMass` returns the real value with a validity flag. It is invalid until a physics body exists, so do not call it during construction.

> Both the component and the visualizer were fixed in 1.5.2. If either seems to do nothing, check your plugin version first.



## SetCenterOfMassOffset

`SetCenterOfMassOffset` adds a further offset on top of whichever base applies, and **sets rather than accumulates**.

That makes it safe to call repeatedly — shifting mass as cargo loads, or as a fuel tank empties, without drift.
