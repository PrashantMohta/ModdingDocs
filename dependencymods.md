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

## TODO

- Add other core mods that are missing
- Add small explanation of each core mod and what it can do
- Add other intermod things like:
  - DebugMod
  - HKMP
  - Rando adding to pools
