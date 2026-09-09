# Legacy Vehicle Setup Guide

## Requirements

- **A Static Mesh or Skeletal Mesh for the chassis and wheels of a vehicle**
  - Static Meshes will need a separate wheel mesh (A basic static mesh body and wheel are included in the plugin for demo purposes, you must check "Show Plugin Content" in the view options to see it)
  - Skeletal Meshes should be setup for or similar to Unreal's _WheeledVehicle_ system
- **Own the Advanced Vehicle System Plugin**
- **A basic understanding of Unreal Engine, some steps in this tutorial will expect you to know how to create or set up things without guidance**

## Step 0: Optional - Configure Project Settings

Consider making the recommended changes from the [Project Settings page](https://overtorque-creations.com/Dev/Docs/#AVS/general/Project_Settings.md), they will dramatically increase physics stability, and top speed.

## Step 1: Configure your Inputs

<!-- side-by-side:41 -->
![Unreal input settings with Handbrake, ShifterUp and ShifterDown action mappings, plus VehicleForward, VehicleBrake and VehicleSteering axis mappings](../assets/images/tutorials-Creating-Vehicles-01.png)
<!-- split -->
Before we can setup the vehicle, we need to know what our inputs are. So head into your Project Settings > Input and set up something similar to this image.
<!-- /side-by-side -->

## Step 2: Create the Base Vehicle

<!-- side-by-side:50 -->
![Pick Parent Class dialog filtered to AVS_, with AVS_Vehicle selected under Pawn > VehicleSystemBase](../assets/images/tutorials-Creating-Vehicles-02.png)
<!-- split -->
A. Create a new blueprint and choose the base class "AVS_Vehicle" as the parent class. I normally name this blueprint "ProjectName_Vehicle" as it will usually include any code/settings/components that you want to share across all of the vehicles in your project.
<!-- /side-by-side -->

B. Open your new vehicle blueprint and add a SpringArm and attach a camera to it. We won't be setting up any inputs to move the camera in this tutorial so I'd recommend rotating the SpringArm to the side slightly so you'll be able to see the wheels as you drive it.

<!-- side-by-side:48 -->
C. Open the Event Graph. The vehicle is not running and in park by default, so for the tutorial vehicle we want to start the engine and move the shifter to drive.

> Note: As of AVS 1.2 there is a new config option for vehicles to spawn with their engines running.
<!-- split -->
![Event BeginPlay wired through Parent BeginPlay into Start Engine and Set Shifter Position set to Drive](../assets/images/tutorials-Creating-Vehicles-03.png)
<!-- /side-by-side -->

<!-- side-by-side:48 -->
D. Also in the event graph, setup your vehicle to use the inputs you made in Step 1
<!-- split -->
![Blueprint graph grouped into Shifter, Brakes and Throttle/Steering sections, wiring input events into the vehicle's input functions](../assets/images/tutorials-Creating-Vehicles-04.png)
<!-- /side-by-side -->

### Step 2b: (Optional) Set up Automatic Shifting

<!-- side-by-side:50 -->
![Vehicle - Transmission category with Automatic Shifter Position enabled alongside Automatic Transmission](../assets/images/tutorials-Creating-Vehicles-05.png)
<!-- split -->
AVS 1.2.5 added an additional option for automatic shifter positions. If you would like to use this feature you will need to slightly change the configuration above.
<!-- /side-by-side -->

<!-- side-by-side:48 -->
A. Open your project input settings and remove the VehicleBrake input. Open your VehicleForward input and add your brake keys with a value of ( -1.0 )
<!-- split -->
![Input axis mapping named VehicleThrottle with W and gamepad right trigger at 1.0, and S and left trigger at -1.0](../assets/images/tutorials-Creating-Vehicles-06.png)
<!-- /side-by-side -->

<!-- side-by-side:48 -->
B. In your event graph from step 2, remove the old brake input. Replace the 'SetThrottleInput' node with the new 'SetThrottleAndBrakeInput' node. This node is required for Automatic Shifting to function correctly.
<!-- split -->
![InputAxis VehicleThrottle wired into a Set Throttle and Brake Input node](../assets/images/tutorials-Creating-Vehicles-07.png)
<!-- /side-by-side -->

## Step 3: Create your Vehicle

<!-- side-by-side:32 -->
![Blueprint context menu with Create Child Blueprint Class highlighted](../assets/images/tutorials-Creating-Vehicles-08.png)
<!-- split -->
A. Right click on your base vehicle blueprint and select "Create Child Blueprint Class", this will be your actual vehicle so you can name it accordingly. For the tutorial I will name mine "Car".
<!-- /side-by-side -->

B. Open your new vehicle blueprint, then select the VehicleMesh component from the list and set it to your desired mesh. AVS comes with a placeholder mesh in it's content. To see it you'll need to have "Show Plugin Content" checked in the content browser, or "Show Engine Content" if you are using UE5.

<!-- side-by-side:48 -->
C. Wheels in AVS are separate components, called "Vehicle_Wheel". Each wheel component holds the data for itself, so you can configure every wheel individually.

You'll need to add one for each wheel you need on the vehicle. Then move the component to the correct location. Naming the components according to their respective location on the vehicle can also help productivity later on.
<!-- split -->
![Components list with four named wheel components and their gizmos positioned under the chassis in the viewport](../assets/images/tutorials-Creating-Vehicles-09.png)
<!-- /side-by-side -->

## Step 4: Configure your Vehicle

A. Select ClassName(self) in the components list or press the "Class Defaults" button, you will notice in the details panel there are 4 categories Prefixed "Vehicle -". These are where you configure values related to the vehicle as a whole. For now we just want to edit the transmission values, so open "Vehicle - Transmission".

<!-- side-by-side:48 -->
B. In the "Gears" section you will see there are 2 gears by default. Gear 0 will always be the reverse gear, and then everything after is a forward gear. Add 2 more gears, then configure them like the following image.

### Explanation of Gear Variables

EndSpeed - Maximum speed of the gear

StartSpeed - Speed at which this gear will be at its maximum torque

UpShift/Downshift - Speed at which the transmission chooses a new gear

MaxTorque - Torque at the StartSpeed of the gear

MinTorque - Torque at the EndSpeed of the gear
<!-- split -->
![Gears array with four elements expanded, showing end speed, start speed, shift points, RPM and torque values for each gear](../assets/images/tutorials-creating-vehicles-pre-ue5-2-10.png)
<!-- /side-by-side -->

<!-- side-by-side:48 -->
C. Now you'll want to configure your wheel components. Select one of your wheels, in the details panel open the Wheel Config section. You can then select your desired wheel mesh if you have one. If no mesh is selected, it will use a sphere collision at the specified wheel radius size. If you do select a mesh, the collision will instead be that of the static mesh you select, so make sure your mesh collision is the shape you intend (a simple sphere is recommended). The following image is setup for a rear wheel drive vehicle.

> Note: if you have rotated the wheel components to have your wheel mesh face the opposite direction you may need to check the invert torque option.
<!-- split -->
![Side-by-side rear and front wheel configs: the rear wheel is a driving and handbrake wheel, the front is a turning wheel with a 30 degree max steering angle](../assets/images/tutorials-creating-vehicles-pre-ue5-2-11.png "Rear wheel drive: drive and handbrake at the rear, steering at the front")
<!-- /side-by-side -->

## Step 5: Setup the config assist HUD

The plugin includes a basic HUD to assist in the creation of vehicles. This step is not required but is recommended if you are not extremely familiar with Unreal Engine.

<!-- side-by-side:48 -->
A. Create both a player controller blueprint and a game mode blueprint.
<!-- split -->
![Pick Parent Class dialog with Player Controller and Game Mode Base circled](../assets/images/tutorials-Creating-Vehicles-12.png)
<!-- /side-by-side -->

<!-- side-by-side:48 -->
B. Open the new player controller, drag off the execution pin on Event Begin Play. Then type "Create Widget" in the box, and press Enter. Set the new Create widget node to the VehicleSetup HUD. Then grab the return value and use it to create the "Add to Viewport" Node. Lastly set the owning player to Self.

> Note 1: In UE4 you'll need to check "Show Plugin Content" in the view options to find the VehicleSetup_HUD
>
> Note 2: In UE5 (if the plugin is installed to the engine) you'll have to check "Show Engine Content" in the content drawer in order to view the plugin content.
<!-- split -->
![Animated walkthrough of creating the Vehicle Setup HUD widget and adding it to the viewport on BeginPlay](../assets/images/tutorials-creating-vehicles-pre-ue5-2-17.gif)
<!-- /side-by-side -->

<!-- side-by-side:40 -->
If you need multiplayer, add a check to see if we are on the local controller.
<!-- split -->
![Event BeginPlay branching on Is Local Player Controller before creating the HUD widget and adding it to viewport](../assets/images/tutorials-Creating-Vehicles-14.png)
<!-- /side-by-side -->

C. Now open the main Unreal tab (the one with your loaded level), and click the blueprints button. You need to set your gamemode to the one you created, and then set your PlayerController within that game mode.

![Two-part screenshot: selecting MyNewGameMode as the GameModeBase class, then selecting MyPlayerController inside it](../assets/images/tutorials-creating-vehicles-pre-ue5-2-15.png "Game mode first, then the player controller inside it")

## Step 6: Test what you have so far

If you have successfully followed along, you should now be able to drive your vehicle. Either set the default pawn in your GameMode to your new vehicle, or place one in your level and set "Auto Possess Player 0" in the details panel.

Once you are in the game, you can use your ShiftUp and ShiftDown inputs to move the shifter, you will see it change on the HUD. However if you're using the "Automatic Shifter Position" feature this will happen automatically.

If you want to use the buttons and sliders, press 'SHIFT + F1' to bring your mouse cursor up. You can also use the HUD to play with different values for your springs and find the best values. The new springs values will be applied to every wheel, overriding whatever you had set before.

![Checker-textured car driving in engine with the vehicle setup HUD showing air speed, gear, torque and spring sliders](../assets/images/tutorials-creating-vehicles-pre-ue5-2-16.png)

---

This point forward is only for Skeletal Mesh Vehicles, you will need to complete Steps 1 through 6 or have a preexisting working vehicle to follow from this point.

## Step 7: Setup your Skeletal Mesh

For this tutorial I am using the Buggy mesh from the Epic Games Vehicle Game project which you can find here: [https://www.unrealengine.com/marketplace/en-US/learn/vehicle-game](https://www.unrealengine.com/marketplace/en-US/learn/vehicle-game)

Whatever Skeletal Mesh you have needs to have the Chassis of the vehicle as the root bone, and 1 bone for each animated wheel. Those are the only requirements.

<!-- side-by-side:48 -->
A. The first thing that needs to be done is to setup the physics asset for your mesh. Open the existing one or create a new one to work with. Then clear everything in the skeleton tree so we are working with a blank asset.

> Note: This image was recorded in Unreal 4.23. Newer versions of the engine show this tree a little differently but the action is the same, select everything in the Physics Asset and hit delete. As long as there are no physics constraints left everything is fine.
<!-- split -->
![Animated walkthrough of clearing all bodies and constraints out of a physics asset](../assets/images/tutorials-creating-vehicles-pre-ue5-2-17.gif)
<!-- /side-by-side -->

B. Now create the collision for your vehicle chassis. If you don't know how to do this you might want to look up some tutorials on setting up a physics asset, as it's outside the scope of this one. I'm going to use a multi convex hull on my vehicle, since this is just a tutorial vehicle and doesn't need to be extremely accurate. Make sure you do not create constraints!

<!-- side-by-side:48 -->
C. Now add some type of collision shape to each wheel bone. You don't need to set these shapes up because they will not be used for collisions. AVS will use these as a handle to control the wheel's position.
<!-- split -->
![Physics asset skeleton tree with B_L_wheelJNT selected and the Add Shape > Add Sphere menu open over a monster truck mesh](../assets/images/tutorials-creating-vehicles-pre-ue5-2-18.png)
<!-- /side-by-side -->

D. We now need to setup our collision mesh, this is going to be a separate static mesh that is going to act as the collision for the vehicle. The reason this needs to be done because our skeletal mesh is only cosmetic and does not contribute to the physics of the vehicle. You will need a dummy static mesh for this, so copy any static mesh to the directory you are keeping your Skeletal Mesh, and name it something like SM_Buggy_Collision. I usually use the Basic Cube mesh that you can spawn in the level editor, but you can use anything.

<!-- side-by-side:48 -->
E. We now need to copy the collisions you made in step 7B. In your PhysicsAsset, select the collision body you made for your Chassis, then in the details panel go to _Body Setup > Primitives_ and copy the whole Primitives section.
<!-- split -->
![Physics asset details panel with the Primitives context menu open on Copy, showing 4 convex elements](../assets/images/tutorials-creating-vehicles-pre-ue5-2-19.png)
<!-- /side-by-side -->

<!-- side-by-side:48 -->
F. Then in your new collision asset (the dummy static mesh), go to Collision > Primitives and paste them. You should now be able to see the Vehicle collision on your Collision mesh, if not click the collision button in the toolbar make sure "Simple Collision" is checked.
<!-- split -->
![Static mesh editor with the pasted green convex hull collision surrounding a plain cube mesh](../assets/images/tutorials-creating-vehicles-pre-ue5-2-20.png)
<!-- /side-by-side -->

## Step 8: Add Skeletal Mesh to Vehicle

A. Open your vehicle's blueprint, set the VehicleMesh component to the collision mesh you created.

<!-- side-by-side:40 -->
B. Add a SkeletalMeshComponent that attaches to the VehicleMesh and set it to your vehicle's Skeletal Mesh. Then attach any wheels that will be animating a bone to your new skeletal mesh component.
<!-- split -->
![Components hierarchy with SkeletalMesh under VehicleMesh and four wheel components under it, with a monster truck in the viewport](../assets/images/tutorials-creating-vehicles-pre-ue5-2-21.png)
<!-- /side-by-side -->

<!-- side-by-side:65 -->
C. Under Rendering for the VehicleMesh, you can set it to never be visible, or be hidden in game only.
<!-- split -->
![Rendering section with Visible checked and Hidden in Game checked](../assets/images/tutorials-creating-vehicles-pre-ue5-2-22.png)
<!-- /side-by-side -->

<!-- side-by-side:40 -->
D. Now select your wheel components, and clear the static mesh in the wheel config if it is using one (This is assuming you want to use the default sphere collisions, you can use a static mesh for collisions if you wish). Check the box "Connect to Bone" and type in the associated bone name. If you typed the bone's name correctly, the component will snap to the bone location. Adjust the Wheel radius to match your wheel size (if you are using the default sphere collision). The rotation however will not snap, so make sure you have the X axis facing in the forward direction, Z axis facing in the upward direction, and the Y axis facing towards the side of the wheel that faces away from the vehicle.
<!-- split -->
![Vehicle blueprint with a wheel component selected and snapped to the monster truck's front wheel](../assets/images/tutorials-creating-vehicles-pre-ue5-2-23.png)
<!-- /side-by-side -->

## Step 9: Drive your Vehicle

You should now be able to test your vehicle. From here you should have an idea of how the system works, play around with the settings to make whatever you want!

![Monster truck driving in engine with the vehicle setup HUD showing 28 mph in gear 2](../assets/images/tutorials-creating-vehicles-pre-ue5-2-24.png)
