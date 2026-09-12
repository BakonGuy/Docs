# Important Information



## Wheel Mesh Collisions

<!-- side-by-side:48 -->
When using **Physics Wheels** (not Raycast Wheels), you must pay careful attention to the shape of the collision on your wheel mesh. AVS supports any collision shape, but **for best results—especially at high speeds—you should use a simple sphere collision.**

Unreal Engine does **not** support true cylinder collisions in Chaos.

Faking a cylinder using convex collision may _look_ correct, but it will not behave correctly—these approximated shapes are not perfectly round, and they will cause the wheels to bounce or behave erratically when driving fast.

While unconventional setups are allowed, **a sphere is the most stable and recommended option** for typical vehicles using physics-based wheels.

**Raycast Wheels are not affected by this limitation**. You can use any collision shape for them, since the collision shape does not influence their simulation behavior.
<!-- split -->
![Wheel with convex collision bouncing erratically at speed compared to a sphere collision](../Assets/Images/general-important-information-01.gif)
<!-- /side-by-side -->



## Blueprint Events No Longer Need a Parent Call

If you used AVS 1.4 or earlier, you had to right click certain Blueprint events and add a call to the parent function, or the vehicle would not work correctly.

**That is no longer necessary.** As of 1.5 the entire plugin is native C++, and Unreal calls the native implementation for you before your Blueprint event runs. Construction Script, BeginPlay, Destroyed, OnPossessed, and Unpossessed all work without any extra wiring.

Parent calls carried over from an older project are harmless and do not need to be removed. New vehicles do not need them at all.

> This is the largest difference when following older tutorials or community posts. If a guide tells you to add a call to the parent function, it was written for 1.4 or earlier.



## Skeletal Mesh Collisions Are Disabled by Default

<!-- side-by-side:57 -->
When you attach a **Skeletal Mesh** to the VehicleMesh component, **AVS will automatically disable its collisions** during construction. This is intentional.

Skeletal meshes cannot weld to the root physics body, and leaving their collisions enabled can lead to unexpected or unstable physics behavior. Since skeletal meshes are typically used for **visual/cosmetic purposes only**, disabling their collision helps prevent these issues and improves performance.

If you **do** want a specific skeletal mesh to keep its collisions active, just add the **KeepCollision** tag to that mesh in the editor. AVS will skip disabling collision for any mesh with that tag.

To turn the behavior off entirely, set **Disable Skeletal Collisions** to false under _Advanced Vehicle System → Physics_.
<!-- split -->
![Details panel showing a Component Tags array with a single KeepCollision entry](../Assets/Images/general-important-information-03.png "Add KeepCollision to Component Tags to opt a mesh out")
<!-- /side-by-side -->



## Avoid Physics Constraints in Skeletal Mesh Physics Assets

<!-- side-by-side:48 -->
If you're using a **Skeletal Mesh** as part of your vehicle setup and need to add **physics constraints**, do **not** place those constraints inside the mesh's Physics Asset—especially if they rely on the skeletal mesh's root body.

Instead, create all such constraints **directly in the vehicle blueprint**, and connect them to the **VehicleMesh** component (the AVS root). This avoids a class of difficult-to-debug issues where constraints behave erratically—typically showing stuttery or jittery motion in parts of the mesh that are meant to simulate.

This behavior stems from how Unreal updates transforms for components that aren't welded to the root. Constraints defined inside skeletal mesh assets may receive outdated transform data, especially when mixing simulated and non-simulated bodies. The result is **inconsistent, unstable physics behavior**, usually visible as violent jittering of constrained parts.

To prevent this entirely, set up all physics constraints externally in your vehicle blueprint where they can reference the true root of the simulation.

A demonstration of this issue can be seen in the video to the right.
<!-- split -->
![Constraint component in a vehicle Blueprint with Component Name 1 set to SK_Mesh and Component Name 2 set to VehicleMesh](../Assets/Images/general-important-information-05.png "Constraints live in the vehicle Blueprint and reference VehicleMesh directly")

[Video: constraint jitter caused by constraints defined inside the Physics Asset](https://drive.google.com/file/d/1qQbSTNSwgUVjdT_krVEZ9wILCqH5RnGQ/preview)
<!-- /side-by-side -->



## Passive Mode and the Tick Event

By default a vehicle enters **passive mode** once it comes to rest. This is a low resource mode that massively improves performance when you have a lot of parked or idle vehicles in a level, and it is why a hundred stationary vehicles cost far less than a hundred moving ones.

The catch is that passive mode gatekeeps the standard Tick event. If you have Blueprint logic on Tick that needs to keep running while the vehicle sits still, it will stop running.

Use **AVS_AlwaysTick** for logic that must run no matter what, or **AVS_PassiveTick** for logic that only matters while resting. Both are covered on the [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md) page.

> Possessed vehicles are already covered: a vehicle controlled by a player or an AI controller never goes passive. The case to watch is a vehicle moved **without** being possessed, such as by a sequencer or your own movement code. For those, set **Allow Passive Mode** to false or override **Determine Passive State**.



## Platform Support

AVS supports every platform Unreal supports.

The plugin's `PlatformDenyList` names **VisionOS**, but that is a packaging requirement rather than a limitation. FAB submission requires a non-empty deny list — without one the block is stripped when the plugin is packaged, and verification fails. VisionOS is listed because it is the least used platform.

Remove it from `VehicleSystemPlugin.uplugin` if you are targeting VisionOS.

> Older guidance said iOS had to be added to a whitelist by hand. That is no longer true — the plugin denies specific platforms rather than allowing specific ones, and iOS needs no edit.



## Installing the Plugin to a Project Instead of the Engine

AVS installs to the engine through the launcher. To have it live inside a project instead — which is what you want for source control, or for a team without the launcher install:

1. Install AVS to a launcher build of the engine as normal.
2. Copy `...\UE_5.x\Engine\Plugins\Marketplace\VehicleSystemPlugin` to `YourProject\Plugins\VehicleSystemPlugin`.

It behaves the same as the engine install, except the project will likely recompile the plugin the first time it opens.