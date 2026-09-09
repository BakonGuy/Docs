# Hitch / Trailers

## Basic Understanding

Trailers in AVS are not their own class. They are standard AVS vehicles that happen to have a receiving hitch on them.

That makes for a versatile workflow, since trailers get access to every feature a normal AVS vehicle has. You could build a trailer that a second player "drives", managing its steering or engine. It also means features added in future updates carry over to your trailers with no extra work.

## Step 1: Create your trailer "vehicle"

<!-- side-by-side:57 -->
Create a new vehicle and set it up like a trailer. Follow the [Quick Start guide](https://overtorque-creations.com/Dev/Docs/#AVS/tutorials/Quick_Start.md) if you need it. The plugin includes a basic trailer mesh and wheel if you don't have your own.
<!-- split -->
![Checker-textured trailer with a single axle and an A-frame tongue, sitting in the Unreal viewport](../assets/images/components-Hitch-01.png "The trailer mesh included with the plugin")
<!-- /side-by-side -->

## Step 2: Add a hitch component to your trailer

<!-- side-by-side:65 -->
Add a hitch component to the trailer. Its location defines where the trailer is towed from, so make sure the X axis (red) faces forward relative to the trailer mesh.

If you're using the trailer mesh included with the plugin, the hitch point is at zero and you won't need to move the component at all.
<!-- split -->
![Add Component menu filtered to "vehicle", showing Vehicle Hitch among the VehicleSystem components](../assets/images/components-Hitch-02.png "Add Component → Vehicle Hitch")
<!-- /side-by-side -->

## Step 2a: Add a collision overlap to your hitch component

<!-- side-by-side:48 -->
Only necessary if you plan to use the **Hitch to Overlapped** feature. It's an easy way to make nearby hitches connect to each other, and it's included because it's the most common case.

Add a collision shape as a child of the hitch component, sized to define the zone you want to hitch from.
<!-- split -->
![Component list with a Sphere collision parented under Vehicle_Hitch, and the sphere shown at the trailer tongue in the viewport](../assets/images/components-Hitch-03.png "A sphere under the hitch defines the overlap zone")
<!-- /side-by-side -->

## Step 3: Configure your hitch

<!-- side-by-side:57 -->
Select the hitch component and open its details panel. This is the receiving hitch, so set **Type** to `Trailer Hitch`.

If it helps, think of `Tow Hitch` as the tow ball and `Trailer Hitch` as the coupler.
<!-- split -->
![Hitch - Config panel with Type set to Trailer Hitch and rotation limits of 45, 70 and 120 degrees](../assets/images/components-Hitch-04.png "Trailer Hitch is the coupler side of the connection")
<!-- /side-by-side -->

## Step 3a: Explanation of connect types

**Connect Types** lets you define different classes of hitch. Two hitches can only connect if they share a connect type.

For example, a trailer could define both `Standard` and `Semi`, letting it hitch to any tow hitch that declares either one.

## Step 4: Add a hitch to your towing vehicle

<!-- side-by-side:57 -->
Open the vehicle you want to tow with and add a hitch component to it. Add a collision to it as in step 2a if you're using overlaps. Leave the type as `Tow Hitch`.

Move the hitch to the towing location, again making sure the X axis (red) points forward.
<!-- split -->
![Hitch component positioned behind the rear bumper of a checker-textured car, with its rotation gizmo visible](../assets/images/components-Hitch-05.png "The tow hitch sits where the ball would be on a real vehicle")
<!-- /side-by-side -->

## Step 5: Hitch Inputs

Now to actually connect the two. In your vehicle Blueprint, add a keyboard event (or whatever input you prefer) and wire up one of these two options.

<!-- side-by-side:50 -->
**Option 1: Hitch to Overlapped**

Runs the hitch function on the first overlapping hitch it finds. Requires a collision volume on each hitch (step 2a).

![Keyboard L event wired into a Hitch to Overlapped node targeting Vehicle Hitch](../assets/images/components-Hitch-06.png)
<!-- split -->
**Option 2: Hitch**

Attempts to hitch two specific hitches regardless of where they are in the world. This does **not** teleport the trailer, so use it carefully.

![Keyboard L event wired into a Hitch node with a Trailer Pawn's Vehicle Hitch supplied as the To pin](../assets/images/components-Hitch-07.png)
<!-- /side-by-side -->

Either way, be mindful of where you call these from. In single player it's safe to call them from any actor at any time. In multiplayer you can only call them from the owning client — the client possessing the pawn, or the server.

## Step 6: Enjoy trailers!

<!-- side-by-side:41 -->
![A checker-textured trailer parked beside a checker-textured car, both rendered in-engine](../assets/images/components-Hitch-08.png)
<!-- split -->
That's all there is to it. I put a lot of effort into making this possible, so I hope you enjoy it. If you have questions or suggestions for this page, message me on the Discord server or by email. Thanks for using AVS!
<!-- /side-by-side -->
