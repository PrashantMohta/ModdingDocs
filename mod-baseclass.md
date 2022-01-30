# The Mod base class

## Introduction

The Mod base class is the class that allows the Modloader (Modding Api) to load your mod into the game, as such, the class deriving from this class is the one that will serve as the entry point of your mod, where you can : 

 - [Set your load priority](#set-your-load-priority)
 - [Set your Mod's Name](#set-your-mods-name)
 - [Set your Mod's Version](#set-your-mods-version)
 - [Request for preloads](#request-for-preloads)
 - [Initialize your mod](#initialize-your-mod)
 
 you can also use additional features of the base class, by implementing optional interfaces
 
  - [Make your mod toggleable ](#make-your-mod-toggleable)
  - [Add Global Settings](#add-global-settings)
  - [Add Save Settings](#add-local-settings)
  - [Add a Mod Menu](#add-a-mod-menu)
 
 ### Set your load priority
By default Mods are loaded in the alphabetical order of their names, if for some reason you need to ensure that your mod loads earlier or later than some other mods, you can use the load priority to determine that.
To do this add the following function to your Mod class :

``` cs
public override int LoadPriority() => PRIORITY;
```
Where `PRIORITY` is an int, higher values of priority will load your mod later and lower values will load your mod earlier in relation to other mods. i.e any Mod with priority 5 loads after any Mods priority 4 or lower

 ### Set your Mods Name
 The Default name of your mod is the name of the Mod class, if you need to change that, you can do so by passing the name to the base class constructor, like so :
 
``` cs 
public MyFirstMod() : base("My 1st Mod") { }
```

 ### Set your Mods Version
 To allow user's to tell your mod versions apart and to allow you to debug issues, you can set the version
 
``` cs 
 public override string GetVersion() => "v0.1";
```

> Note: it is advisable to always change your version before publishing a new release to make sure that people are able to use the right version when installing / reporting bugs.

 ### Request for preloads
 
The Mod base class allows your mod to request the Modding Api to preload game objects from a particular scene.
for more information see  [Preloading game objects](https://prashantmohta.github.io/ModdingDocs/preloads.html) .

 ### Initialize your mod
The Mod base class allows your mod to [receive the Preloaded objects](https://prashantmohta.github.io/ModdingDocs/preloads.html), and work with setting up your mod code in Initialize. This method is called after your mod class is constructed, but beware that it is called more than once, especially if your mod is toggled on and off.

## Optional Interfaces

### Make your Mod toggleable

>Todo

### Add global settings

Global settings are the way mods can save settings that should persist across save games

>Todo 

### Add local settings

Local settings are the way mods can save data per save game, this may be settings but for more involved mods this may include saving play state for example : if a player has acquired a custom item or not in this save, could be saved here.

>Todo

### Add a mod menu

Mod menu is the way for mods to give the player an interactive UI to change their settings as opposed to editing settings files manually. Instead of using custom UI to handle this, it is recommended to use the Mod menu so that players can have a consistent experience and the pause menu is not cluttered, wherever possible.

for more about creating menus, head over to the dedicated page that covers the different types of [Mod menus](https://prashantmohta.github.io/ModdingDocs/modmenu.html)
