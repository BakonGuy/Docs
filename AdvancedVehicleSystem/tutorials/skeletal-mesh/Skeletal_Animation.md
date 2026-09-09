# Skeletal Mesh AnimBP

How to feed wheel data from AVS into your AnimBP, using the UE5 template offroad car as an example.

> Using the **Connect to Bone** feature instead? You want the [Skeletal Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/tutorials/skeletal-mesh/Skeletal_Wheels.md) page.

## Step 1: Create your AnimBP

<!-- side-by-side:57 -->
Create a new AnimBP using the skeleton you want to animate.
<!-- split -->
![Content browser context menu on SKM_Offroad_Skeleton with Create > Anim Blueprint highlighted](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-01.png)
<!-- /side-by-side -->

## Step 2: Setting up the Event Graph

Select the **Event Graph** tab inside your new AnimBP.

<!-- side-by-side:41 -->
![Variables list with a boolean Init and four vectors named BL, BR, FL and FR](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-02.png "One vector per wheel, plus a boolean for the init state")
<!-- split -->
### 2A — Variables

Create a variable for each wheel you need data from.
<!-- /side-by-side -->

<!-- side-by-side:41 -->
![Event Blueprint Update Animation and Try Get Pawn Owner feeding a Cast To Vehicle_UE5_Buggy node](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-03.png)
<!-- split -->
### 2B — Cast to actor

Cast to your vehicle actor.
<!-- /side-by-side -->

<!-- side-by-side:41 -->
![Cast output wired into an Is Valid node checking the Vis Wheel BR component](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-04.png)
<!-- split -->
### 2C — Validate a wheel component

After casting, validate one of your wheel components.

During construction the actor cast passes, but the components haven't been constructed yet — so for a single frame, reading data from the wheel components throws errors. The validate step stops the event graph from continuing until the wheels exist.
<!-- /side-by-side -->

<!-- side-by-side:58 -->
![Get Initialization State on the AVS vehicle feeding a SET node for the Init boolean](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-05.png)
<!-- split -->
### 2D — Save vehicle initialization state

Save the initialization state. You'll use it later to stop the graph animating pre-runtime.
<!-- /side-by-side -->

<!-- side-by-side:58 -->
![Four Get Wheel Mesh nodes feeding Get World Location into SET nodes for BL, BR, FL and FR](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-06.png "One chain per wheel: wheel mesh, world location, store it")
<!-- split -->
### 2E — Save data from wheels

With access to the wheels, save whatever data your Anim Graph needs.
<!-- /side-by-side -->

Put together, the finished event graph looks like this:

![The complete event graph: Update Animation, Try Get Pawn Owner, cast, Is Valid check, initialization state, and four wheel location chains](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-07.png "The whole event graph in one shot")

## Step 3: Setting up the Anim Graph

Select the **Anim Graph** tab inside your new AnimBP.

<!-- side-by-side:58 -->
![Four Transform (Modify) Bone nodes in series, one per wheel bone, each taking its saved translation vector](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-09.png)
<!-- split -->
### 3A — Transform wheel bones

Transform each of your bones as needed. Here we set bone location in world space, so set the transform node's settings to match.
<!-- /side-by-side -->

![Transform (Modify) Bone details with Bone to Modify PhysWheel_BL, Translation Mode Replace Existing and Translation Space World Space](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-08.png "Replace Existing in World Space — the translation comes straight from the wheel")

<!-- side-by-side:41 -->
![Component To Local feeding a Control Rig node with Alpha 1.0](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-10.png)
<!-- split -->
### 3B — Offroad control rig

If you're following along with the UE5 template offroad car, or you have a control rig of your own, add it here. This example uses the offroad control rig unmodified.
<!-- /side-by-side -->

<!-- side-by-side:58 -->
![Blend Poses by bool driven by the Init variable, choosing between the animated pose and the mesh space ref pose](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-11.png "Init picks between the live pose and the reference pose")
<!-- split -->
### 3C — Only animate on init

Prevent the anim graph from running until the vehicle has passed initialization. This keeps the mesh in its default pose during initialization and in the editor — useful if you want to snap wheels to bone locations, which is how the offroad car is set up in the AVS demo.
<!-- /side-by-side -->

The finished anim graph:

![The complete anim graph: ref pose into four Transform (Modify) Bone nodes, through Component To Local and Control Rig, blended by Init into the Output Pose](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-12.png)

## Step 4: Test

Set the AnimBP on the skeletal mesh inside your vehicle Blueprint. If everything is wired up correctly, your vehicle should animate during play.

![Offroad buggy with independent suspension articulating over a yellow ramp](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-13.png)
