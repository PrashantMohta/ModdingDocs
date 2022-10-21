---
title: Dependency Mods
nav_order: 13
has_children: true
---
# Dependency Mods

There are many dependency mods that were created to make modding easier. Below you can find a list of them.
> Note: All descriptions are added by the mod author unless specified.

## [Satchel](https://github.com/PrashantMohta/Satchel)

A Library mod by Dandy and Mulhima that aims to make modding more accessible by providing simpler ways of achieving complex functionality, also has some Utilities for general things.
[Features and documentation](Satchel/Satchel.md)

## [SFCore](https://github.com/SFGrenade/SFCore)

A core mod made by SFGrenade.  
Some of the core features are:

- Helpers: Easier ways to simply add miscellaneous content.
  - E.g. achievements, charms, environment particles, items, menu styles and title logos.
  - Provides events and methods that allows you to add your content.
- Generic mod classes
  - This abstracts code used to have global and save settings.
- ModInterOp
  - This allows mods to register methods with a name so other mods can call them without having a hard dependency.
- MonoBehaviours
  - These are useful for making custom scenes.
  - Abstract generation of Hollow Knight's MonoBehaviours for simple use.
    - Some of these require editor scripts to be easily usable.
- Util
  - Miscellaneous utils like FsmUtil, MiscCreator, SpriteUtil, USceneUtil and Util.

## [Vasi](https://github.com/fifty-six/HollowKnight.Vasi)

A core mod made by 56

## [FrogCore](https://github.com/RedFrog6002/FrogCore)

A core mod made by RedFrog

## [MenuChanger](https://github.com/homothetyhk/HollowKnight.MenuChanger)

A core mod made by Homothety

## [ItemChanger](https://github.com/homothetyhk/HollowKnight.ItemChanger)

A core mod made by Homothety. 
Relevant Links:
- [Documentation](https://homothetyhk.github.io/HollowKnight.ItemChanger/)

## [MagicUI](https://github.com/BadMagic100/HollowKnight.MagicUI)

A core mod made by BadMagic that simplifies creating non-menu, in-game UI elements. Unity's UI elements are low-level components, meaning you need to be
very familiar with their implementation details to piece them together into something useful and usually requiring a significant amount of boilerplate code.
In contrast, MagicUI provides:
* A hierarchical layout tree composed only of various "arrangable elements," which encapsulate boilerplate code and repeated arrangement behaviors.
* Implicit re-measure/re-arrange support - simply change a property and the minimum necessary subset of the UI will be reflowed to accomodate the new value.
* Several useful built-in element types, including buttons, text entry, text display, and images.
* Simple API to implement custom element types for specific use cases
* TextureLoader API to easily load images from embedded resources.

Relevant Links:
- [Documentation](https://badmagic100.github.io/HollowKnight.MagicUI/)
- [Example](https://github.com/BadMagic100/HollowKnight.MagicUI/blob/master/MagicUIExamples)

## [HKCore FSMUtil](https://github.com/hk-modding/HK.Core.FsmUtil)

A core mod made meant to replace all different PlayMakerExtensions, FsmExtensions, FsmUtil and FSMHelper in the long run.

## [HKTool](https://github.com/HKLab/HollowKnightMod.Tool)

A core mod made by HKLab

## [Osmi](https://github.com/Clazex/HollowKnight.Osmi)

A core mod made by Clazex that aims to provide convenient utilities and allow the user to get rid of writing commonly used code snippets repeatedly.

Features include but not limited to:

- `CharmUtil` and `Charm` enum: provide a easy-to-maintain way to manage checking and (un)equipping for charms without magic numbers
- `Dict` and `LangUtil`: automatic localization loading, switching and getting
- `RandomSelector`: simulation of `SendRandomEventV3`, but way more flexible
- `Refs`: short-hand for singleton instances
- `SimpleFSM`: a straightforward approach to define a finite state machine from scratch with `IEnumerator`s, attributes and full access to `MonoBehaviour` functionalities, also interops with vanilla `PlayMakerFSM`
- `Osmi.FsmActions` namespace: convenient `PlayMakerFSM` actions

... and a large number of extension method utilities

## [HKMirror](https://github.com/TheMulhima/HKMirror)

A core mod that makes it easier to access PlayerData, use reflection and use on/il hooks.   
#### Explanation of features:
* **PlayerDataAccess** 
  * A class that allows you to access PlayerData as properties while also using best practices (i.e. going through PlayerData.Get/Set functions). 
  * Can make your code from `PlayerData.instance.SetInt(nameof(PlayerData.health), PlayerData.instance.GetInt(nameof(PlayerData.health) - 1));` to `PlayerDataAccess.health -= 1`
  * How to use: add a `using HKMirror;` and get/set fields the same way you would do with `PlayerData.instance.` but instead use `PlayerDataAccess.`
* **HKMirror.Reflection** 
  * HKMirror does all the reflection for you, allowing you to call any private field/method in almost any class effortlessly.
  * How to use: add a `using HKMirror.Reflection;` and call the extension method `.Reflect()` on any instance of the class you want to have publicised access to. For example: `gameObject.GetComponent<HealthManager>().Reflect().enemyType = 6` (enemy type is normally a private field). For convenience, common classes like HeroController and GameManager can be accessed as `HeroContollerR.DoAttack()` with a `using HKMirror.Reflection.SingletonClasses;` 
* **HKMirror.Hooks** 
  * The On Hooks contain all on hooks that are available in the `On` namespace plus hooks for methods that aren't there such as API generated methods and property getter/setters.
    * How to use: add a `using HKMirror.Hooks.OnHooks` and find the class you want, in this example GameManager. The class in HKMirror will be called OnGameManager and there are 3 subclasses in it, BeforeOrig, AfterOrig, WithOrig. BeforeOrig are events which are invoked before orig(self) is called in an onhook. AfterOrig are events which are invoked after orig(self) is called in an onhook. In these, you can access the parameters through accessing the fields of the passed in `args` (e.g. `args.self`). An example would be `OnGameManager.BeforeOrig.IncreaseGameTimer += args => Log(args.timer);`. On the other hand, WithOrig work like, and are structured like normal On Hooks
  * The IL hooks contain all correct IL Hooks (i.e gives the GetStateMachineTarget version of IEnumerators ILs that run after every yeild return as opposed to regular IL Hooks) and includes ILHooks that aren't in the IL namespace such as API generated functions and property getters/setters.
    * How to use: add a `using HKMirror.Hooks.ILHooks` and find the class you want, in this example HeroContoller. The class in HKMirror will be called ILHeroController in which there are the il hooks you can hook onto.

#### Available namespaces:
* HKMirror
  * PlayerDataAccess
  * HKMirror.Reflection
    * HKMirror.Reflection.SingletonClasses - Contains publicised access to common classes like HeroController and GameManager
    * HKMirror.Reflection.StaticClasses - Contains publicised access to static classes
    * HKMirror.Reflection.InstanceClasses - Contains publicised access to instance classes
  * HKMirror.Hooks
    * HKMirror.Hooks.OnHooks - Contains the on hook events to hook onto
    * HKMirror.Hooks.ILHooks - Contains the il hook events to hook onto


## TODO

- Add other core mods that are missing
- Add small explanation of each core mod and what it can do
- Add other intermod things like:
  - DebugMod
  - HKMP
  - Rando adding to pools
