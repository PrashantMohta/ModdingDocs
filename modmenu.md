---
nav_order: 10
---
# ModMenu

## Introduction 
Starting with Hollow Knight 1.5.x.x, the Modding API added the ability for mods to add a menu to the "Mods" menu in game.This is a standard and preferred way to add GUI options to configure your mod's settings. The Modding API has 2 ways to create a Modmenu, either by implementing the `IMenuMod` interface or the `ICustomMenuMod` interface.

## IMenuMod
`IMenuMod` allows you to add a list of horizontal selections, that allow you to choose 1 option each from a list. This is a common and versatile element used across HK menus for configurations. IMenuMod is really simple to implement but it is limited in what it can do.

[Official Documentation](https://hk-modding.github.io/api/articles/menu_api.html#the-imenumod-interface)

## ICustomMenuMod 
`ICustomMenuMod` allows you to add a bunch of different menu elements. It is more complex to implement, but offers a lot more flexibility than IMenuMod.

[Official Documentation](https://hk-modding.github.io/api/articles/menu_api.html#icustommenumod-and-the-menubuilder-api)

## Satchel.BetterMenus
`Satchel.BetterMenus` is built to allow mods to implement `ICustomMenuMod` functionality while offering the simplicity of IMenuMods.
It also makes a bunch of commonly desired features like updating the menu dynamically, showing/hiding elements simple to use.

[Documentation](Satchel/BetterMenus/better-menus.md)
