---
title: "UE Programming"
description: "Unreal Engine programming C++ and Blueprints."
lead: "Unreal Engine programming C++ and Blueprints."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu:
  docs:
    parent: "unreal"
weight: 30
toc: true
---

## Source control

Working with big assets can be challenging, here are some options:

* [Git Plugin](https://github.com/ProjectBorealis/UEGitPlugin). GitHub LFS is not free.
* [PlasticSCM](https://www.plasticscm.com/). Includes free plan.
* [Helix Perforce](https://www.perforce.com/products/helix-core).

## Build max parallel actions

Modify or create:
  * Linux:
    ```
    $HOME/.config/Unreal Engine/UnrealBuildTool/BuildConfiguration.xml
    ```
  * Windows:
    ```
    mydocuments\Unreal Engine\UnrealBuildTool\BuildConfiguration.xml
    ```

BuildConfiguration.xml contents:

```xml
<?xml version="1.0" encoding="utf-8" ?>

<Configuration xmlns="https://www.unrealengine.com/BuildConfiguration">

<ParallelExecutor>
  <MaxProcessorCount>30</MaxProcessorCount>
  <ProcessorCountMultiplier>2</ProcessorCountMultiplier>
  <!-- Free memory per action in bytes, used to limit the number of
    parallel actions if the machine is memory starved.
    Set to 0 to disable free memory checking.
    PROCEED WITH CAUTION -->
  <MemoryPerActionBytes>0</MemoryPerActionBytes>
</ParallelExecutor>

</Configuration>
```

In the previous example, by setting **ProcessorCountMultiplier to 2**, the build process takes into account logical threads (if your CPU supports *Simultaneous Multi Threading (SMT)* or *hyper-threading*) to calculate the max actions to execute in parallel. Then, setting **MaxProcessorCount to 30** limits the compile thread count to 30. So if your CPU has 16 physical cores and 2 logical cores each (32 in total), this leaves 2 logical cores entirely free for other software, resulting in a smoother experience when multitasking during project compilation. *This can change depending on your available RAM if you do not set **MemoryPerActionBytes** to 0, but doing that can eventually crash your system.*

If your settings have been configured properly, you will get the following UE build process output:

```
Building XXX actions with 30 processes...
```

More info:
 * [Source](https://gpuopen.com/learn/threadripper-for-gamedev-ue4/)
 * [Oficial link](https://docs.unrealengine.com/5.1/en-US/build-configuration-for-unreal-engine/) see ParallelExecutor section.



## Unreal Plugins

* [Sample project](https://github.com/UEplugins/TextAsset)
* [Crash course](https://learn.unrealengine.com/course/2436528)


## UE Audio

### Reverb

To activate the reverb, add the following line to your project **/Config/DefaultEngine.ini**:

```
[Audio]
UseAudioMixer=true
```

Create a **Sound > Sound Attenuation** asset and configure it on your **Cue** as the attenuation preset. Add an **Audio Volume** to your level, set it big enough to cover your playable area and set the following properties:

* Settings -> Apply Reverb: True.
* Settings -> Reverb effect: select one from the dropdown.

* For more info check this video [Reverb Tutorial](https://www.youtube.com/watch?v=ZFjD5xtoMSg). Note that if you added the configuration to your project .ini, you won't need to execute additional commands on the UE console.

## UE Animation

Documentation: [Animation](https://docs.unrealengine.com/en-US/Engine/Animation/index.html)

* [Basics Course](https://www.youtube.com/watch?v=-slbsR_WWAE&list=PLL0cLF8gjBpqpCGt9ayn4Ip1p6kvgXYi2)
* [UE Livestream](https://www.youtube.com/watch?v=YVC-DL9Ibf0)
  * Animation Blueprints
  * Locomotion
  * Blendspaces
  * Anim montages
  * Caching poses
  * Layering
  * Modifying bones
  * Look at
* Other contents:
  * [Subanim](https://docs.unrealengine.com/en-US/Engine/Animation/AnimHowTo/SubAnimInstance/index.html)
  * [Anim offset](https://docs.unrealengine.com/en-US/Engine/Animation/AnimHowTo/AimOffset/index.html)
  * [Physical anims](https://www.youtube.com/watch?v=N1tDjbFXeOo)
  * [Paragon](https://www.youtube.com/watch?v=YlKA22Hzerk)
* [How to's](https://docs.unrealengine.com/en-US/Engine/Animation/AnimHowTo/index.html)

## UE AI

[Official docs](https://docs.unrealengine.com/en-US/Engine/ArtificialIntelligence/index.html)

* To enable the debugger set a key binding in *Project Settings > Engine - Gameplay Debbuger > Activation Key*. If you're using a non default US keyboard or a keyboard with international bindings, it is recommended to set a plain key as Asterisk (*) or Minus (-), instead of something like an Apostrophe (') that won't be triggered by a single key-press on a US-International setup.

### Navmeshes

* RecastNavMesh: update some navmesh settings on the scene. Also update-able from the project settings window searching for the option *Agent* (you can add support for multiple types of agents from here).

* Navmesh modifier volume: Adds additional information to the navmesh. Exmample: high cost areas, not allowed areas, etc.

* To disable Auto-updates: Editor Preferences -> Level Editor -> Misc -> Update Navigation Automatically.

* **More info:** [Official docs](https://docs.unrealengine.com/en-US/Resources/ContentExamples/NavMesh/index.html)

### AI Perception

[Official docs](https://docs.unrealengine.com/en-US/Engine/ArtificialIntelligence/AIPerception/index.html)

* Add component **AIPerception**.

* [UE AI Perception with C++](https://www.thinkandbuild.it/UE-ai-perception-system/)

### Behavior trees

[Official docs](https://docs.unrealengine.com/en-US/Engine/ArtificialIntelligence/BehaviorTrees/index.html)

* [Quick Start Guide](https://docs.unrealengine.com/en-US/Engine/ArtificialIntelligence/BehaviorTrees/BehaviorTreeQuickStart/index.html)

* **Nodes**: [Official reference](https://docs.unrealengine.com/en-US/Engine/ArtificialIntelligence/BehaviorTrees/BehaviorTreeNodeReference/index.html)
  * **Composite Nodes**: These are the nodes that define the root of a branch and the base rules for how that branch is executed.
  * **Task Nodes**: These are the leaves of the Behavior Tree, these nodes are the actionable things to do and don't have an output connection.
  * **Decorator Nodes**: Also known as conditionals. These attach to another node and make decisions on whether or not a branch in the tree, or even a single node, can be executed.
  * **Service Nodes**: These attach to Composite nodes, and will execute at their defined frequency as long as their branch is being executed. These are often used to make checks and to update the Blackboard. These take the place of traditional Parallel nodes in other Behavior Tree systems.

### Environment Query System (EQS)

[Official Docs](https://docs.unrealengine.com/en-US/Engine/ArtificialIntelligence/EQS/index.html)

### UE AI movement C++

See this tutorial: https://www.vikram.codes/blog/ai/01-basic-navigation
Improved and updated code to seek for a Random Reachable Location in the Navmesh:

* [MovementAIController.h](https://gist.github.com/dacanizares/6f47164e3d86d9fd6d19bd129d888695)
* [MovementAIController.cpp](https://gist.github.com/dacanizares/5db9c59281a9c9049bf819acce7e29bc)
* Don't forget to add the NavigationSystem and AIModule modules in your {project}.Build.cs. [More info](https://docs.unrealengine.com/en-US/Programming/Modules/Gameplay/index.html): it would be suffice to add "NavigationSystem" and "AIModule" strings to the PublicDependencyModuleNames.AddRange(...) parameters.


## How to do it?

* **Convert Mouse Location to World Space**: Interact with 3D objects using players' mouse.
  [See video](https://www.youtube.com/watch?v=b1_efR9hrT4)
* **Reducing Packaged Game Size**: [See here](https://docs.unrealengine.com/en-US/Engine/Performance/ReducingPackageSize/index.html)
* **More about Unreal C++**: [See UE C++ section]({{< relref "ue-cpp" >}})
