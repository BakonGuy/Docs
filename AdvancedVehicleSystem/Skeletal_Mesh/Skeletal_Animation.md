# Skeletal Mesh AnimBP

How to animate a skeletal mesh's wheel and suspension bones from AVS wheels, with an AnimBP. The example is the Offroad Car from Unreal's vehicle template.

This works with raycast and physics wheels, with the wheels as separate meshes. To have physics wheels move the bones directly, without an AnimBP, see [Skeletal Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Skeletal_Mesh/Skeletal_Wheels.md).



## How It Works

The AVS wheels and their meshes do the driving. The skeletal mesh is only for looks.

Each frame, the AnimBP's event graph reads where each AVS wheel mesh is and stores it. The anim graph then moves the matching bones to those positions.



## Step 1: Create the AnimBP

<!-- side-by-side:57 -->
Right-click the skeletal mesh's skeleton and choose **Create → Anim Blueprint**.
<!-- split -->
![Content browser context menu on SKM_Offroad_Skeleton with Create > Anim Blueprint highlighted](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-01.png)
<!-- /side-by-side -->



## Step 2: Read the Wheel Positions in the Event Graph

<!-- side-by-side:41 -->
![Variables list with a boolean Init and four vectors named BL, BR, FL and FR](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-02.png "One vector per wheel, plus a boolean for the initialization state")
<!-- split -->
1. Create a vector variable for each wheel, such as `FL`, `FR`, `BL` and `BR`, and a boolean named `Init`.
<!-- /side-by-side -->

<!-- side-by-side:41 -->
![Event Blueprint Update Animation and Try Get Pawn Owner feeding a Cast To Vehicle_UE5_Buggy node](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-03.png)
<!-- split -->
2. On **Event Blueprint Update Animation**, get **Try Get Pawn Owner** and cast it to your vehicle class.
<!-- /side-by-side -->

<!-- side-by-side:41 -->
![Cast output wired into an Is Valid node checking the Vis Wheel BR component](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-04.png)
<!-- split -->
3. Check one of the vehicle's wheel components with **Is Valid**, and stop if it is not valid. While the vehicle is being constructed, the cast succeeds before the wheel components exist, and reading them would give errors for one frame.
<!-- /side-by-side -->

<!-- side-by-side:58 -->
![Get Initialization State on the AVS vehicle feeding a SET node for the Init boolean](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-05.png)
<!-- split -->
4. Store the vehicle's **Get Initialization State** in `Init`. It is true once AVS has finished setting the vehicle up.
<!-- /side-by-side -->

<!-- side-by-side:58 -->
![Four Get Wheel Mesh nodes feeding Get World Location into SET nodes for BL, BR, FL and FR](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-06.png "One chain per wheel: wheel mesh, world location, store it")
<!-- split -->
5. For each wheel, call **Get Wheel Mesh**, then **Get World Location**, and store the result in that wheel's variable.
<!-- /side-by-side -->

The finished event graph:

![The complete event graph: Update Animation, Try Get Pawn Owner, cast, Is Valid check, initialization state, and four wheel location chains](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-07.png "The whole event graph")



## Step 3: Move the Bones in the Anim Graph

<!-- side-by-side:58 -->
![Four Transform (Modify) Bone nodes in series, one per wheel bone, each taking its saved translation vector](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-09.png)
<!-- split -->
1. Add a **Transform (Modify) Bone** node for each wheel bone, in a chain, and connect each one's **Translation** to that wheel's variable.
2. On each node, set **Translation Mode** to **Replace Existing** and **Translation Space** to **World Space**.
<!-- /side-by-side -->

![Transform (Modify) Bone details with Bone to Modify PhysWheel_BL, Translation Mode Replace Existing and Translation Space World Space](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-08.png "Replace Existing in World Space")

<!-- side-by-side:41 -->
![Component To Local feeding a Control Rig node with Alpha 1.0](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-10.png)
<!-- split -->
3. If the mesh has a Control Rig, such as the template Offroad Car's, add it after the bone nodes. This example uses the Offroad Car's Control Rig unchanged.
<!-- /side-by-side -->

<!-- side-by-side:58 -->
![Blend Poses by bool driven by the Init variable, choosing between the animated pose and the mesh space ref pose](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-11.png "Init picks between the live pose and the reference pose")
<!-- split -->
4. Add a **Blend Poses by bool** driven by `Init`. Use the animated pose when true, and the reference pose when false.

Until the vehicle is set up, and in the editor, the mesh then stays in its default pose. The demo's Offroad Car relies on this to snap its wheel components to the bone locations.
<!-- /side-by-side -->

The finished anim graph:

![The complete anim graph: ref pose into four Transform (Modify) Bone nodes, through Component To Local and Control Rig, blended by Init into the Output Pose](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-12.png)



## Step 4: Test the Vehicle

Set the skeletal mesh component in your vehicle Blueprint to use the new AnimBP, then press Play. The wheel bones and suspension should follow the AVS wheels.

![Offroad buggy with independent suspension articulating over a yellow ramp](../Assets/Images/Skeletal_Mesh/Skeletal_Animation/tutorials-skeletal-mesh-skeletal-animation-13.png)



## Next Steps

- [Skeletal Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Skeletal_Mesh/Skeletal_Wheels.md): physics wheels moving the bones directly.
- [Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md): wheel modes, suspension and grip.
