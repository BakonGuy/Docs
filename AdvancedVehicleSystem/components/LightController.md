---
title: "Vehicle Lights"
source: https://avs.overtorque-creations.com/components/LightController
---

# Vehicle Lights

## **Creating Lights for your Vehicle**

### **Understanding the Basics**

Lights in the Vehicle System are controlled by multiple _Vehicle _LightController_ components within your vehicle. Every vehicle has a list of _Groups_ that the Light Controllers use to determine the correct intensity. These groups can be thought of as Booleans, they are either on or off. Lights placed as subcomponents of the light controllers will have their Intensity setting driven automatically using the relationship settings defined within their respective controller. When a group is activated, each light controller containing that group will loop through its relationships, setting all of its lights intensities to the highest active group on the hierarchy. This effectively allows us to create complex setups where every light on the vehicle can be driven by any amount of variables.

### **Looking at an Example**

![Image](../assets/images/components-LightController-01.png)

This is the muscle car from the demo project (can be found on the home page).

When you turn your headlights on in a real car usually the blinker lights will turn on as well, but at a low intensity. The muscle car is recreating this by defining a relationship with the _HeadLights_ group, setting the intensity of the light to 10 if that group is active.

This means that when the head lights are on, the blinker will also activate, but since its the lowest on the hierarchy, the _BlinkerLeft_ group will override it when it is also active.

Now imagine you have some code that clicks the blinker on and off, by toggling the _BlinkerLeft_ group. When the blinker group is inactive the blinker will set itself to 0, but if the headlights are on, it would set it to 10. Therefore mimicking a real blinker perfectly.

### **Controlling Light Groups**

![Image](../assets/images/components-LightController-02.png)

Controlling the Light Groups is super easy. You can activate and deactivate groups with the following blueprint nodes within the Vehicle.

- **ToggleLightsActive**
- **SetLightsActive**
- **GetLightsActive**
- **UpdateLights** (Applies your changed Light Groups)

_Note:_You will need to call the **UpdateLights** function after changing your Light Groups, this is for performance reasons as you might want to change multiple light groups at once.

## **Vehicle Decorations, driven by lights**

![Image](../assets/images/components-LightController-03.png)

When your Vehicle Lights are updating, the **UpdateLightDecorations** event is called once per Light Controller. This is meant to be used for extra effects you want to apply to the vehicle, such as updating materials on the vehicle mesh or playing sounds. You can also just override the **UpdateLights** function if you want to update everything at once instead, just be sure to call the parent function.

The Light Controllers have 2 functions you can use to determine how to update your decorations:

**GetItensity**, and **HasActiveLights**

![Image](../assets/images/components-LightController-04.png)

