# Skeletal Mesh Wheels (Connect to Bone)

How to drive wheels inside a skeletal mesh with AVS's physics wheel mode, using the **Connect to Bone** feature.

> Using separated wheel meshes — which raycast wheels require? You want the [Skeletal Animation](https://overtorque-creations.com/Dev/Docs/#AVS/tutorials/skeletal-mesh/Skeletal_Animation.md) page instead.

## Step 1: Set up your skeletal mesh

This tutorial uses the Sports Car from the UE5 Vehicle Template.

<!-- side-by-side:48 -->
### 1A — Prepare the physics asset

Set up the physics asset for your mesh. Open the existing one or create a new one, then **clear all physics constraints** from it.
<!-- split -->
![Animated walkthrough of clearing the bodies and constraints out of the sports car's physics asset](../../assets/images/tutorials-skeletal-mesh-skeletal-wheels-01.gif)
<!-- /side-by-side -->

### 1B — Create the chassis collision

Create collision for the vehicle chassis if it doesn't have any. Setting up a physics asset is outside the scope of this guide, so look up a tutorial if you need one. This example uses a multi convex hull, since it's a tutorial vehicle and doesn't need to be especially accurate.

> Make sure you do not create constraints.

<!-- side-by-side:48 -->
### 1C — Add a collision shape to each wheel bone

Add some kind of collision shape to each wheel bone if they don't already have one. You don't need to configure these shapes — they're never used for collision. AVS uses them as a handle to control the wheel's position.

If you can't see your wheel bones in the skeleton tree, open settings and enable **Show All Bones**.
<!-- split -->
![Physics asset editor with Phys_Wheel_FR selected in the skeleton tree and the Add Shape > Add Sphere menu open](../../assets/images/tutorials-skeletal-mesh-skeletal-wheels-02.png "Add Shape → Add Sphere on each wheel bone")
<!-- /side-by-side -->

### 1D — Get a collision mesh

Vehicle simulation needs a collision mesh to act as the chassis. The skeletal mesh is purely cosmetic here. If you already have a suitable Static Mesh for the chassis, use it and skip the rest of step 1.

If you don't have one:

- Copy a Static Mesh you want to use as the chassis and paste it into the same directory as your skeletal mesh — for example `SM_Buggy_Collision`.
- The collision mesh will not be rendered, so any static mesh works. A cube is a good choice.

<!-- side-by-side:48 -->
### 1E — Copy the chassis primitives

Now copy the collision you made in step 1B. In your physics asset, select the collision body for the chassis, then in the details panel go to _Body Setup → Primitives_ and copy the whole Primitives section.
<!-- split -->
![Physics asset details panel with the Primitives context menu open on Copy](../../assets/images/tutorials-skeletal-mesh-skeletal-wheels-03.png "Copy the entire Primitives section")
<!-- /side-by-side -->

<!-- side-by-side:32 -->
### 1F — Paste them onto the collision mesh

In your collision mesh, go to _Collision → Primitives_ and paste. You should now see the vehicle collision on the collision mesh. If you don't, click the collision button in the toolbar and make sure **Simple Collision** is checked.
<!-- split -->
![Static mesh editor showing a cube mesh wrapped in the pasted green car-shaped collision hull, with Simple Collision enabled](../../assets/images/tutorials-skeletal-mesh-skeletal-wheels-04.png "The car-shaped hull pasted onto a plain cube mesh")
<!-- /side-by-side -->

## Step 2: Add the skeletal mesh to your vehicle

### 2A — Set the vehicle mesh

Open your vehicle's Blueprint and set the `VehicleMesh` component to your collision mesh.

<!-- side-by-side:40 -->
### 2B — Attach the skeletal mesh

Add a Skeletal Mesh component attached to `VehicleMesh` and set it to your vehicle's skeletal mesh. Then attach any wheels that will animate a bone to that new skeletal mesh component.
<!-- split -->
![Components hierarchy with SkeletalMesh under Vehicle Mesh, and Wheel_BL, Wheel_BR, Wheel_FL and Wheel_FR under the skeletal mesh](../../assets/images/tutorials-skeletal-mesh-skeletal-wheels-05.png "Wheels parent to the skeletal mesh, not the collision mesh")
<!-- /side-by-side -->

<!-- side-by-side:57 -->
### 2C — Hide the collision mesh

Under **Rendering** for `VehicleMesh`, set it to never be visible, or hidden in game only.
<!-- split -->
![Rendering section with Visible checked and Hidden in Game checked](../../assets/images/tutorials-creating-vehicles-pre-ue5-2-22.png)
<!-- /side-by-side -->

### 2D — Configure the wheels

- Select your wheel components. Pick a static mesh if you want one; leave it blank and a sphere collision is used instead.
- Make sure you're using **Physics** wheel mode.
- Check **Connect to Bone** and type the associated bone name. If the name is correct, the component snaps to the bone location.
- Rotation does *not* snap, so make sure the X axis faces forward and the Z axis faces up.
- Adjust the wheel radius to match your wheel size.

![Vehicle Wheel - Config with Wheel Mode set to Physics, Wheel Radius 39 cm, Connect to Bone checked and Bone Name Phys_Wheel_FR](../../assets/images/tutorials-skeletal-mesh-skeletal-wheels-07.png "Connect to Bone plus the exact bone name is all it takes")

## Step 3: Drive your vehicle

You should now be able to test. If everything is working, each wheel bone follows the location of its respective wheel component.

![Sports car driving in engine with the vehicle setup HUD showing 21 mph, drive, gear 1 and four wheels](../../assets/images/tutorials-skeletal-mesh-skeletal-wheels-08.png)

## Step 4: Using AnimBPs with this setup

Because of how AVS simulates wheels in this setup, wheel bone locations don't update inside the AnimBP. You can still read the actual bone locations from the AnimBP's event graph — so save your wheel locations there, then use them in the anim graph.

<!-- side-by-side:50 -->
![AnimBP event graph: Initialize Animation storing the owning component as Buggy Mesh, then Update Animation reading four socket transforms into FL, FR, BR and BL](../../assets/images/tutorials-skeletal-mesh-skeletal-wheels-09.png "Get Socket Transform per wheel, stored for the anim graph")
<!-- split -->
### Event Graph

On initialization, get a reference to the mesh in the world. This example saves it as `BuggyMesh`, a Skeletal Mesh Component variable.

In the **Update Animation** event, use that saved mesh to read the current bone locations and store them. There are four wheels here, saved under names close to their bone names.
<!-- /side-by-side -->

<!-- side-by-side:40 -->
### Anim Graph

With those variables saved from the event graph, the anim graph can use them in place of the non-working bone locations.
<!-- split -->
![AnimGraph with four Look At nodes, one per suspension bone, each driven by a saved wheel location vector](../../assets/images/tutorials-skeletal-mesh-skeletal-wheels-10.png "Look At nodes aiming each suspension bone at its saved wheel location")
<!-- /side-by-side -->

### Notes

This AnimBP ships with the demo project as part of the Unreal Buggy vehicle, under the name `VH_BuggyAnimBP_VS`. If you run into trouble, digging through that content is the fastest way to see a working setup.
