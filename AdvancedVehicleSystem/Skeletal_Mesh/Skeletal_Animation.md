# Skeletal Mesh AnimBP

This page provides information on how to input wheel data from AVS into your AnimBP, using the UE5 Template Offroad Car as an example. If you are using the "Connect to Bone" feature of AVS you will need the [Skeletal Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Skeletal_Mesh/Skeletal_Wheels.md) page instead.



## Step 1: Create Your AnimBP

<!-- side-by-side:57 -->
Create a new AnimBP using the skeletion you wish to animate
<!-- split -->
![Content browser context menu on SKM_Offroad_Skeleton with Create > Anim Blueprint highlighted](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-01.png)
<!-- /side-by-side -->



## Step 2: Setting up the Event Graph

Select the Event Graph tab inside your new AnimBP

<!-- side-by-side:41 -->
![Variables list with a boolean Init and four vectors named BL, BR, FL and FR](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-02.png "One vector per wheel, plus a boolean for the init state")
<!-- split -->
### 2A - Variables

Create variables for each wheel you need data from
<!-- /side-by-side -->

<!-- side-by-side:41 -->
![Event Blueprint Update Animation and Try Get Pawn Owner feeding a Cast To Vehicle_UE5_Buggy node](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-03.png)
<!-- split -->
### 2B - Cast to Actor

Cast to your Vehicle Actor
<!-- /side-by-side -->

<!-- side-by-side:41 -->
![Cast output wired into an Is Valid node checking the Vis Wheel BR component](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-04.png)
<!-- split -->
### 2C - Validate a wheel component

After casting, you need to validate one of your wheel components. This is because during construction the actor cast will pass, but the components have not been constructed yet, meaning a single frame will give errors if we attempt to read data from our wheel components. The validate step will prevent the event graph from continuing if the wheels have not been constructed yet.
<!-- /side-by-side -->

<!-- side-by-side:58 -->
![Get Initialization State on the AVS vehicle feeding a SET node for the Init boolean](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-05.png)
<!-- split -->
### 2D - Save vehicle initalization state

Next we will save the initialization state, we will use this later to prevent the graph from animating pre-runtime.
<!-- /side-by-side -->

<!-- side-by-side:58 -->
![Four Get Wheel Mesh nodes feeding Get World Location into SET nodes for BL, BR, FL and FR](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-06.png "One chain per wheel: wheel mesh, world location, store it")
<!-- split -->
### 2E - Save data from wheels

Now that we have access to the wheels. You can save whatever data you need from them for your Anim Graph
<!-- /side-by-side -->

The finished event graph:

![The complete event graph: Update Animation, Try Get Pawn Owner, cast, Is Valid check, initialization state, and four wheel location chains](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-07.png "The whole event graph in one shot")



## Step 3: Setting up the Anim Graph

Select the Anim Graph tab inside your new AnimBP

<!-- side-by-side:58 -->
![Four Transform (Modify) Bone nodes in series, one per wheel bone, each taking its saved translation vector](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-09.png)
<!-- split -->
### 3A - Transform Wheel Bones

Transform each of your bones as needed. Here we will be setting the bone location in world space, make sure to set the transform node's settings accordingly.
<!-- /side-by-side -->

![Transform (Modify) Bone details with Bone to Modify PhysWheel_BL, Translation Mode Replace Existing and Translation Space World Space](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-08.png "Replace Existing in World Space")

<!-- side-by-side:41 -->
![Component To Local feeding a Control Rig node with Alpha 1.0](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-10.png)
<!-- split -->
### 3B - Offroad Control Rig

If your following along using the UE5 template offroad car, or if you have a control rig. You will want to add the control rig here. I used the offroad control rig unmodified.
<!-- /side-by-side -->

<!-- side-by-side:58 -->
![Blend Poses by bool driven by the Init variable, choosing between the animated pose and the mesh space ref pose](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-11.png "Init picks between the live pose and the reference pose")
<!-- split -->
### 3C - Only animate on Init

Now we want to prevent the anim graph from running unless the vehicle has passed initialization. We do this to ensure the mesh is in the default pose during initialization, or in the editor. This can be useful if we want to snap wheels to the bone location, for example, which is how the offroad car is setup in the AVS demo.
<!-- /side-by-side -->

The finished anim graph:

![The complete anim graph: ref pose into four Transform (Modify) Bone nodes, through Component To Local and Control Rig, blended by Init into the Output Pose](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-12.png)



## Step 4: Test

Set the AnimBP on your skeletal mesh inside your vehicle bp, if everything is setup correctly, your vehicle should now animate during play.

![Offroad buggy with independent suspension articulating over a yellow ramp](../Assets/Images/tutorials-skeletal-mesh-skeletal-animation-13.png)
