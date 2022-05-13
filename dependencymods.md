---
nav_order: 13
---
# Dependency Mods

There are many dependency mods that were created to make modding easier. Below you can find a list of them.
> Note: All descriptions are added by the mod author unless specified.

## [Satchel](https://github.com/PrashantMohta/Satchel)

A core mod made by Dandy and Mulhima.  
Some of the core features are:

- BetterMenus: An easier way to implement ICustomMenuMod.
  - Only asks the user to return a list of all the menu elements they want while the menu builder and such are abstracted away.
  - Provides events and methods that allows you to edit/hide elements at runtime.
  - See [example](https://github.com/PrashantMohta/Satchel/tree/master/BetterMenus/Example)
- Others (todo).

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

A core mod made by BadMagic.
Relevant Links:
- [Documentation](https://badmagic100.github.io/HollowKnight.MagicUI/)
- [Example](https://github.com/BadMagic100/HollowKnight.MagicUI/blob/master/MagicUIExamples)

## [HKCore FSMUtil](https://github.com/hk-modding/HK.Core.FsmUtil)

A core mod made meant to replace all different PlayMakerExtensions, FsmExtensions, FsmUtil and FSMHelper in the long run.

## TODO

- Add other core mods that are missing
- Add small explanation of each core mod and what it can do
- Add other intermod things like:
  - DebugMod (Adding to keybinds and such)
  - Rando adding to pools?
