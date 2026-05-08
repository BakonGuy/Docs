---
title: "Skeletal Mesh AnimBP"
source: https://avs.overtorque-creations.com/tutorials/skeletal-mesh/skeletal-animation
---

# Skeletal Mesh AnimBP

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-01.png)

## **Skeletal Mesh **AnimBP**

This page provides information on how to input wheel data from AVS into your AnimBP, using the UE5 Template Offroad Car as an example. If you are using the "Connect to Bone" feature of AVS you will need the [Skeletal Wheels](https://avs.overtorque-creations.com/tutorials/skeletal-mesh/skeletal-wheels) page instead.

## **Step 1: Create Your AnimBP**

Create a new AnimBP using the skeletion you wish to animate

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-02.png)

## **Step**2: Setting up the Event Graph**

Select the Event Graph tab inside your new AnimBP

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-03.png)

### **2A - Variables**

Create variables for each wheel you need data from

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-04.png)

### **2 **B**-**Cast to Actor**

Cast to your Vehicle Actor

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-05.png)

### **2 **C**-**Validate a wheel component**

After casting, you need to validate one of your wheel components. This is because during construction the actor cast will pass, but the components have not been constructed yet, meaning a single frame will give errors if we attempt to read data from our wheel components. The validate step will prevent the event graph from continuing if the wheels have not been constructed yet.

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-06.png)

### **2 **D**-**Save vehicle initalization state**

Next we will save the initialization state, we will use this later to prevent the graph from animating pre-runtime.

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-07.png)

### **2 **E**-**Save data from wheels**

Now that we have access to the wheels. You can save whatever data you need from them for your Anim Graph

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-08.png)

## **Step**3**: Setting up the **Anim **Graph**

Select theAnimGraph tab inside your new AnimBP

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-09.png)

### **3 **A -**Transform Wheel Bones**

Transform each of your bones as needed. Here we will be setting the bone location in world space, make sure to set the transform node's settings accordingly.

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-10.png)

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-11.png)

### **3 **B**-**Offroad Control Rig**

If your following along using the UE5 template offroad car, or if you have a control rig. You will want to add the control rig here. I used the offroad control rig unmodified.

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-12.png)

### **3 **C**-**Only animate on Init**

Now we want to prevent the anim graph from running unless the vehicle has passed initialization. We do this to ensure the mesh is in the default pose during initialization, or in the editor. This can be useful if we want to snap wheels to the bone location, for example, which is how the offroad car is setup in the AVS demo.

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-13.png)

## **Step**4: Test**

Set the AnimBP on your skeletal mesh inside your vehicle bp, if everything is setup correctly, your vehicle should now animate during play.

![Image](../../assets/images/tutorials-skeletal-mesh-skeletal-animation-14.png)

