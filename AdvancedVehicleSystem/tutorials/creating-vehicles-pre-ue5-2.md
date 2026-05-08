---
title: "Legacy Vehicle Setup Guide"
source: https://avs.overtorque-creations.com/tutorials/creating-vehicles-pre-ue5-2
---

# Legacy Vehicle Setup Guide

## **Legacy **Vehicle Setup Guide**

### **Requirements**

- **A Static Mesh or Skeletal Mesh for the chassis and wheels of a vehicle **Static Meshes will need a separate wheel mesh
- Static Meshes will need a separate wheel mesh
- Skeletal Meshes should be setup for or similar to Unreal's _WheeledVehicle_ system
- Skeletal Meshes should be setup for or similar to Unreal's _WheeledVehicle_ system
- **Own the Advanced Vehicle System Plugin**
- **A basic understanding of Unreal Engine, some steps in this tutorial will expect you to know how to create or set up things without guidance**

## **Step 0: Optional - Configure Project Settings**

Consider making the recommended changes from the [Project Settings page](https://avs.overtorque-creations.com/general/project-settings), they will dramatically increase physics stability, and top speed.

## **Step 1: Configure your Inputs**

![Image](../assets/images/tutorials-Creating-Vehicles-01.png)

Before we can setup the vehicle, we need to know what our inputs are. So head into your Project Settings > Input and set up something similar to this image.

## **Step 2: Create the Base Vehicle**

![Image](../assets/images/tutorials-Creating-Vehicles-02.png)

A. Create a new blueprint and choose the base class “AVS_Vehicle” as the parent class. I normally name this blueprint “_ProjectName__Vehicle” as it will usually include any code/settings/components that you want to share across all of the vehicles in your project.

B. Open your new vehicle blueprint and add a SpringArm and attach a camera to it. We won't be setting up any inputs to move the camera in this tutorial so I’d recommend rotating the SpringArm to the side slightly so you’ll be able to see the wheels as you drive it.

C. Open the Event Graph. The vehicle is not running and in park by default, so for the tutorial vehicle we want to start the engine and move the shifter to drive.

_Note: As of AVS 1.2 there is a new config option for vehicles to spawn with their engines running._

![Image](../assets/images/tutorials-Creating-Vehicles-03.png)

D. Also in the event graph, setup your vehicle to use the inputs you made in Step 1

![Image](../assets/images/tutorials-Creating-Vehicles-04.png)

### **Step 2b: (Optional) Set up Automatic Shifting**

![Image](../assets/images/tutorials-Creating-Vehicles-05.png)

AVS 1.2.5 added an additional option for automatic shifter positions. If you would like to use this feature you will need to slightly change the configuration above.

A. Open your project input settings and remove the VehicleBrake input. Open your VehicleForward input and add your brake keys with a value of ( -1.0 )

![Image](../assets/images/tutorials-Creating-Vehicles-06.png)

B. In your event graph from step 2, remove the old brake input. Replace the 'SetThrottleInput' node with the new 'SetThrottleAndBrakeInput' node. This node is required for Automatic Shifting to function correctly.

![Image](../assets/images/tutorials-Creating-Vehicles-07.png)

## **Step 3: Create your Vehicle**

![Image](../assets/images/tutorials-Creating-Vehicles-08.png)

A. Right click on your base vehicle blueprint and select “Create Child Blueprint Class”, this will be your actual vehicle so you can name it accordingly. For the tutorial I will name mine “Car”.

B. Open your new vehicle blueprint, then select the VehicleMesh component from the list and set it to your desired mesh. AVS comes with a placeholder mesh in it's content. To see it you'll need to have "Show Plugin Content" checked in the content browser, or "Show Engine Content" if you are using UE5.

C. Wheels in AVS are separate components, called "Vehicle _Wheel". Each wheel component holds the data for itself, so you can configure every wheel individually.

You'll need to add one for each wheel you need on the vehicle. Then move the component to the correct location. Naming the components according to their respective location on the vehicle can also help productivity later on.

![Image](../assets/images/tutorials-Creating-Vehicles-09.png)

## **Step 4: Configure your Vehicle**

A. Select ClassName(self) in the components list or press the “Class Defaults” button, you will notice in the details panel there are 4 categories Prefixed “Vehicle -”. These are where you configure values related to the vehicle as a whole. For now we just want to edit the transmission values, so open “Vehicle - Transmission”.

B. In the “Gears” section you will see there are 2 gears by default. Gear 0 will always be the reverse gear, and then everything after is a forward gear. Add 2 more gears, then configure them like the following image.

### **Explanation of Gear Variables**

_EndSpeed_- Maximum speed of the gear _StartSpeed_- Speed at which this gear will be at its maximum torque

_UpShift/Downshift_- Speed at which the transmission chooses a new gear

_MaxTorque_- Torque at the StartSpeed of the gear _MinTorque_- Torque at the EndSpeed of the gear

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-10.png)

C. Now you’ll want to configure your wheel components. Select one of your wheels, in the details panel open the Wheel Config section. You can then select your desired wheel mesh if you have one. If no mesh is selected, it will use a sphere collision at the specified wheel radius size. If you do select a mesh, the collision will instead be that of the static mesh you select, so make sure your mesh collision is the shape you intend (a simple sphere is recommended). The following image is setup for a rear wheel drive vehicle._Note: if you have rotated the wheel components to have your wheel mesh face the opposite direction you may need to check the invert torque option._

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-11.png)

## **Step 5: Setup the config assist HUD**

_The plugin includes a basic HUD to assist in the creation of vehicles. This step is not required but is recommended if you are not extremely familiar with Unreal Engine._

A. Create both a player controller blueprint and a game mode blueprint.

![Image](../assets/images/tutorials-Creating-Vehicles-12.png)

B. Open the new player controller, drag off the execution pin on Event Begin Play. Then type “Create Widget” in the box, and press Enter. Set the new Create widget node to the VehicleSetup HUD. Then grab the return value and use it to create the “Add to Viewport” Node. Lastly set the owning player to Self.

_Note 1: In UE4 you'll need to check “Show Plugin Content” in the view options to find the VehicleSetup _HUD_

_Note 2: In UE5 (if the plugin is installed to the engine) you'll have to check "Show Engine Content" in the content drawer in order to view the plugin content._

![Image](../assets/images/tutorials-Creating-Vehicles-13.gif)

If you need multiplayer, add a check to see if we are on the local controller.

![Image](../assets/images/tutorials-Creating-Vehicles-14.png)

C. Now open the main Unreal tab (the one with your loaded level), and click the blueprints button. You need to set your gamemode to the one you created, and then set your PlayerController within that game mode.

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-15.png)

## **Step 6: Test what you have so far**

If you have successfully followed along, you should now be able to drive your vehicle. Either** set the default pawn in your GameMode** to your new vehicle, or** place one in your level and set “Auto Possess Player 0” in the details panel.**

Once you are in the game, you can use your ShiftUp and ShiftDown inputs to move the shifter, you will see it change on the HUD. However if you're using the "Automatic Shifter Position" feature this will happen automatically.

If you want to use the buttons and sliders, press**‘SHIFT + F1’**to bring your mouse cursor up. You can also use the HUD to play with different values for your springs and find the best values. The new springs values will be applied to every wheel, overriding whatever you had set before.

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-16.png)

## **Step 7: Setup your Skeletal Mesh**

For this tutorial I am using the Buggy mesh from the Epic Games Vehicle Game project which you can find here:[https://www.unrealengine.com/marketplace/en-US/learn/vehicle-game](https://www.unrealengine.com/marketplace/en-US/learn/vehicle-game)

Whatever Skeletal Mesh you have needs to have the Chassis of the vehicle as the root bone, and 1 bone for each animated wheel. Those are the only requirements.

A. The first thing that needs to be done is to setup the physics asset for your mesh. Open the existing one or create a new one to work with. Then clear everything in the skeleton tree so we are working with a blank asset.

_Note: This image was recorded in Unreal 4.23. Newer versions of the engine show this tree a little differently but the action is the same, select everything in the Physics Asset and hit delete. As long as there are no physics constraints left everything is fine._

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-17.gif)

B. Now create the collision for your vehicle chassis. If you don’t know how to do this you might want to look up some tutorials on setting up a physics asset, as it’s outside the scope of this one. I’m going to use a multi convex hull on my vehicle, since this is just a tutorial vehicle and doesn't need to be extremely accurate._**Make sure you do not create constraints!**_

C. Now add some type of collision shape to each wheel bone. You don't need to set these shapes up because they will not be used for collisions. AVS will use these as a handle to control the wheel's position.

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-18.png)

D. We now need to setup our collision mesh, this is going to be a separate static mesh that is going to act as the collision for the vehicle._The reason this needs to be done because our skeletal mesh is only cosmetic and does not contribute to the physics of the vehicle._You will need a dummy static mesh for this, so copy any static mesh to the directory you are keeping your Skeletal Mesh, and name it something like _SM_Buggy _Collision_. I usually use the Basic Cube mesh that you can spawn in the level editor, but you can use anything.

E. We now need to copy the collisions you made in step 7B. In your PhysicsAsset, select the collision body you made for your Chassis, then in the details panel go to _Body Setup > Primitives_ and copy the whole Primitives section.

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-19.png)

F. Then in your new collision asset (the dummy static mesh), go to Collision > Primitives and paste them. You should now be able to see the Vehicle collision on your Collision mesh, if not click the collision button in the toolbar make sure “Simple Collision” is checked.

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-20.png)

## **Step 8: Add Skeletal Mesh to Vehicle**

A. Open your vehicle’s blueprint, set the _VehicleMesh_ component to the collision mesh you created.

B. Add a _SkeletalMeshComponent_ that attaches to the _VehicleMesh_ and set it to your vehicle’s Skeletal Mesh. Then attach any wheels that will be animating a bone to your new skeletal mesh component.

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-21.png)

C. Under Rendering for the VehicleMesh, you can set it to never be visible, or be hidden in game only.

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-22.png)

D. Now select your wheel components, and clear the static mesh in the wheel config if it is using one (This is assuming you want to use the default sphere collisions, you can use a static mesh for collisions if you wish). Check the box “Connect to Bone” and type in the associated bone name. If you typed the bone’s name correctly, the component will snap to the bone location. Adjust the Wheel radius to match your wheel size (if you are using the default sphere collision). The rotation however will not snap, so make sure you have the X axis facing in the forward direction, Z axis facing in the upward direction, and the Y axis facing towards the side of the wheel that faces away from the vehicle.

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-23.png)

## **Step 9: Drive your Vehicle**

You should now be able to test your vehicle. From here you should have an idea of how the system works, play around with the settings to make whatever you want!

![Image](../assets/images/tutorials-creating-vehicles-pre-ue5-2-24.png)

