myo-ue4
=======

A [Thalmic Myo](https://www.thalmic.com/en/myo/) Plugin for Unreal Engine 4.

[![GitHub release](https://img.shields.io/github/release/getnamo/myo-ue4.svg)](https://github.com/getnamo/myo-ue4/releases)
[![Github All Releases](https://img.shields.io/github/downloads/getnamo/myo-ue4/total.svg)](https://github.com/getnamo/myo-ue4/releases)

[Main discussion thread](https://forums.unrealengine.com/showthread.php?37876-Plugin-Myo)

# Modern rewrite (v0.9)

Documentation for v0.9 is still pending. Basics: add a ```UMyoControllerComponent``` to your actor of choice and subscribe to events you're interested in. From there you will get ```FMyoControllerData``` filled with all kinds of information. Cast ```FMyoControllerData``` to ```UMyoController``` to call functions on it such as vibrate device and enable raw emg stream.

## Setup

1. Ensure you have the latest [Myo Connect](https://www.myo.com/start) and [firmware](https://developer.thalmic.com/downloads) installed.
2. Download [latest plugin release](https://github.com/getnamo/myo-ue4/releases)
3. Browse to your project (typically found at *Documents/Unreal Project/{Your Project Root}*)
4. Copy *Plugins* folder from the .7z download into your Project root.
5. Launch your project
6. Plugin is now ready to use.






# Old documentation

Latest plugin is bound to beta 9, requires Myos to have [firmware 1.5](https://developer.thalmic.com/downloads) or later.

## Quick Setup
1. Create new or open a project. 
2. Browse to your project (typically found at *Documents/Unreal Project/{Your Project Root}*)
3. Copy *Plugins* folder into your Project root.
4. Copy *Binaries* folder into your Project root.
6. Restart the Editor and open your project again.
7. Select Window->Plugins. Click on Installed and you should see a category called Input and a plugin called Myo Plugin now available. Select Enabled. The Editor will warn you to restart, click restart.
8. When your project has reloaded, the plugin should be enabled and ready to use. 

## How to Use
The plugin is designed with an event driven architecture through a delegate interface. You can access device events through the UE4 Input Mapping system, adding myo support to any blueprint through the Myo Component and Myo Interface, the convenience Blueprint classes provided, or through C++. C++ supports both subclassing of provided example class or by inheriting the MyoDelegate, you can extend your own class to support Myo events. Additionally callable functions support polling for latest data.

## Input Mapping

 1.	For a good example start with a template project.
 2.	Select Edit->Project Settings.
 3.	Select Engine->Input
 4.	Under Action Mappings and Axis Mappings expand the category you wish to add controller movement to. For example if you want to add Forward motion in the standard 3rd person template, click the + sign in MoveForward.
 5.	Change None to the binding you want and adjust the scale to fit. If for example you wanted this to happen when you pitch your arm you would select Myo Orientation Pitch with a scale of say 3.0 to have snappier controls.
 6.	Play and test your scaling adjust as needed.

(Optional) Use key and axis events in any input derived class blueprint (such as controller). Note note that any events you override will cause Engine->Input mapping to stop working for that bind.

Note that only the last paired myo emits input mapping events.

#### *Input Axis Events and Buttons Available*

```
//Poses
FKey MyoPoseRest;
FKey MyoPoseFist;
FKey MyoPoseWaveIn;
FKey MyoPoseWaveOut;
FKey MyoPoseFingersSpread;
FKey MyoPoseDoubleTap;
FKey MyoPoseUnknown;

//Axis
FKey MyoAccelerationX;
FKey MyoAccelerationY;
FKey MyoAccelerationZ;

FKey MyoOrientationPitch;
FKey MyoOrientationYaw;
FKey MyoOrientationRoll;

FKey MyoGyroX;
FKey MyoGyroY;
FKey MyoGyroZ;
```

### Calibration

If you're using only delta values, you may wish to use the raw values instead. If you would like to use absolute orientations (e.g. copying an arm orientation, or getting acceleration in component space) however you may wish to use the calibrated arm space values.

Myo reports values in myo space, which may be rotated around your wrist and has no fixed reference to which direction your screen is. In order to use arm space data emitted from On Arm Moved correctly, your myo should be calibrated. This is done by pointing your myo forward toward your screen or another known reference direction (adjustable in the function) and calling Calibrate Arm Orientation on the myo you wish to calibrate. After this is done all your arm space orientation, gyro, and acceleration data will be automatically rotated to compensate for any wrist roll offset (myo can be arbitrarily rotated around your wrist), yaw offset (difference between screen and magnetic north) and you can safely use these outputs to drive your 3d data in component space.

example of how to calibrate:

<img src="http://i.imgur.com/yQcd5NH.png">

Note that you should perform a 'Sync Gesture' first in order to establish which arm the myo is on and which direction it is facing (toward wrist/arm), calibration picks this up and automatically adjusts for it as well, but if your arm isn't detected (synced), your movements may be inverse if you have the device on the opposite direction compared to default.

### Specific Arm

The myo can distinguish your arms after you've done the [sync gesture](https://support.getmyo.com/hc/en-us/articles/200755509-How-to-perform-the-sync-gesture). There are two methods used in the plugin to get your arm possession.

One, you can get the arm possession by dragging off the myo controller pointer you get from an event such as OnPair or OnPose.

For Example if you had a pose and you wanted it to only affect a particular arm you could do something like this

<img src="http://i.imgur.com/cWjHCPd.png">

Two, if you want a reference to specific arms outside an event, you can get them by dragging off of the myo component and using any of the convenience functions

<img src="http://i.imgur.com/W8vqfa8.png">

Remember that all the available functions/properties relevant to the myo are easily searched by dragging off of the Myo Controller or Myo Component and typing 'myo'. 

### Locking Policy
Since myo beta 7, the SDK implements a locking policy. This means you can perform a double-tap pose on your myo to unlock and make a gesture, which then locks itself automatically again. By default this plugin has no locking policy, if you wish to use one however simply set it at an appropriate time e.g. in your begin play.

<img src="http://i.imgur.com/2x0lZpG.png">


### Raw Data Streams
Since 0.7.7 the plugin supports raw streams. You have to first enable the raw streams for your myo

e.g. using the double tap pose

<img src="http://i.imgur.com/OacMVYW.png">

or using a key event (note that this targets your primary myo since we don't have a myo pointer emitted with a key event)

<img src="http://i.imgur.com/XMC9Ftz.png">

Then subscribing to the OnEmgData event with the optional content library function

<img src="http://i.imgur.com/DsAzDlz.png">

Yields the following results in the rolling template (with Debug Orientation shown and Debug Poses printed to log)

<img src ="http://i.imgur.com/moXdgn8.gif">

To do something useful with the stream, break the struct which encapsulates an int array, one for each of the 8 data streams. The value range for each int is -127 to 128 (int8)

<img src="http://i.imgur.com/GfH30ke.png">

#### *Blueprint Events Available*

<img src="http://i.imgur.com/h49rgju.png">

#### *Blueprint Callable Functions Available*

##### *Myo Component/Convenience Class*

<img src="http://i.imgur.com/1kncm52.png">

##### *Myo Controller*

<img src="http://i.imgur.com/WU3Dumv.png">

#### Example
For example you can show a debug myo orientation like this

<img src="http://i.imgur.com/zrd95U2.png">

with a Draw Orientation function like this (note that this function is included in the optional Content Myo Utility BP Library)

<img src="http://i.imgur.com/aNBewsH.png">

which gives the following result in the Rolling template

<img src="http://i.imgur.com/lVdgzWA.png">

Note in this example two myos were paired and the orientation was obtained from calibrated values thus needed a calibration call to each myo when the user was pointing them toward the screen (e.g. point to screen and bind calibration to make fist).

### Convenience Myo Blueprint Library
Since 0.7.7 the plugin includes optional Content such as the Myo Utility BP Library

<img src="http://i.imgur.com/uV37oa4.png">

To see this content simply select View->Show Plugin Content from your Content browser. Below are a few examples of using this function library.

Debug your orientation

<img src="http://i.imgur.com/9fVTYB8.png">

Draw debug arrows representing each of the 8 raw streams

<img src="http://i.imgur.com/DsAzDlz.png">

or print them out instead using the respective function.

## Events through Blueprint - Convenience Classes
1. Select Window->Class Viewer.
2. Search for "MyoPluginActor"
3. Right click the actor and Create a new Blueprint e.g. "MyoPluginActorBP"
4. Select Graph in the upper right hand corner and right click in the graph to bring up the function search
5. Typing in "myo" will narrow the events down to plugin related.
6. Add your desired events and linkup to your desired functions 
7. Add the created blueprint to the scene (it's an actor subclass) and hit Play.

e.g. If you want to get the acceleration data from your Myo/s add the Event "On Accelerometer Data". Right click again in an empty space in the BP graph and add a function call to "Print String", connect acceleration to string (a conversion node will automatically be made) and drag exec (the white triangle on node) from the event to the function to connect the calls. 

Compile and Play to see the accelerometer data stream as printed output after the myo/s automatically connect and pair.

## Events through C++

#### *Simple Version*
1. Extend or Subclass MyoPluginActor and Override functions you wish to subscribe to e.g. 

```virtual void onPose(int32 myoId, uint64 timestamp, int32 pose) override;```


#### *Extend your own Class to Receive Events*
1. Ensure your project has "MyoPlugin" added to your PublicDependencyModuleNames in your *{Project}.build.cs*
2. Add a ```UMyoControllerComponent``` to your actor of choice
3. [Bind your listening functions to the multicast delegates](https://docs.unrealengine.com/en-us/Programming/UnrealArchitecture/Delegates/Multicast) available in the component. You most likely are interested in ```OnPoseChanged``` and ```OnArmMoved```.

To poll for the latest data, get your desired Myo Controller either via listening to the pairing event or getting your primary myo e.g.

```c++
UMyoControllerComponent* Component;	//instantiated somewhere
UMyoController* PrimaryController = Component->PrimaryMyo();
```

then you can access the data directly via

```c++
PrimaryController.MyoData;
```

with the available data defined in [MyoController.h](https://github.com/getnamo/myo-ue4/blob/master/Source/MyoPlugin/Public/MyoController.h#L7)

All functionality follows how you use it in blueprint, check source files for specific method names.

## Shipping/Packaged Builds
Projects require code, if you are using a blueprint only project, add an empty class and compile your project module. You simply do File->Add Code to Project and it can be anything so I usually just pick None->Create Class and then it will ask you to open visual studio where you just hit compile (Build solution). If you haven't added code before follow the unreal engine <a href="https://docs.unrealengine.com/latest/INT/Programming/QuickStart/1/index.html">programming Quick Start guide</a>. Essentially it boils down to downloading the free Visual Studio Community and changing a few small configs.

## Community
* Point any questions and queries to the [plugin unreal engine thread](https://forums.unrealengine.com/showthread.php?37876-Plugin-Myo) or this repository's [issues page](https://github.com/getnamo/myo-ue4/issues)

Anything not covered by [Myo SDK](https://developer.thalmic.com/docs/api_reference/platform/legal-notices.html) and [Unreal Engine EULA](https://www.unrealengine.com/eula) in <span property="dct:title">Myo UE4 Plugin</span> is covered by MIT license.

[![license](https://img.shields.io/github/license/getnamo/myo-ue4.svg)](https://github.com/getnamo/myo-ue4/blob/master/LICENSE)

