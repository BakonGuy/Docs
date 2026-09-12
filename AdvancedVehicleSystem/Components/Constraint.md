# Constraint

`AVS_Constraint` is a thin wrapper around Unreal's Physics Constraint component. It behaves the same way, with the constraint categories kept visible and the irrelevant ones hidden, plus one convenience function.



## Basic Understanding

Use it for what you would use any physics constraint for — doors, hoods, tailgates, suspension arms, tow ropes. Anything that should move under physics while staying attached to the vehicle.

The reason to use this component rather than Unreal's own is placement. Constraints belong in the vehicle Blueprint, constrained against `VehicleMesh`, and not inside a skeletal mesh's Physics Asset.



## AVS Constraint vs Physics Constraint

<!-- side-by-side:57 -->
**It keeps constraints out of the Physics Asset.** Constraints defined inside a skeletal mesh's Physics Asset receive outdated transform data and jitter violently — the problem described in [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md). Building them in the vehicle Blueprint against `VehicleMesh` avoids it entirely, and this component is the natural place to do that.

**It matches the rest of the plugin.** AVS uses this class internally for wheels and hitches, so anything you build with it behaves consistently.
<!-- split -->
![Constraint component in a vehicle Blueprint with Component Name 1 and 2 set to a door mesh and VehicleMesh](../Assets/Images/_placeholder.png "Constrain to VehicleMesh, not to a skeletal mesh bone")
<!-- /side-by-side -->



## Adding a Constraint

1. Add the component to the vehicle and position it at the joint — the hinge line for a door, the pivot for an arm.
2. Set **Component Name 1** and **Component Name 2** to the two things being joined. One should normally be `VehicleMesh`.
3. Set your linear and angular limits. For a door, that is free swing on one angular axis and locked on everything else.
4. Set breakable thresholds if the part should come off under enough force.

From there it is a standard Unreal physics constraint, so existing knowledge and tutorials apply.



## Soft Linear Constraints

`SetLinearSoftConstraint(bool SoftConstraint, float Stiffness, float Damping)` sets the linear soft constraint parameters in one call.

A soft linear constraint allows the joint to move slightly under load instead of holding rigidly. Use it for a tow rope, a soft engine mount, or anything that should absorb a shock instead of transmitting it into the chassis.

Higher **Stiffness** resists displacement more. Higher **Damping** settles movement faster. A rope needs low stiffness and moderate damping; a mount needs both higher.
