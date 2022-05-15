---
title: BluePrints
parent: BetterMenus
grand_parent: Dependency Mods
---
# BluePrints
Blueprints is a class that has been created to help provide and ease the creation of commonly used functionality in modmenus.

## Key And Button Bind
A blueprint to create a row with keybind and a button bind where only the binding area of them are selectable
```cs
MenuRef = new Menu("Example Mod Menu", new Element[]
        {
            Blueprints.KeyAndButtonBind("Action 1", keyBinds.Key1, keyBinds.Button1),
        });
```
![Key and Button Bind Example](/ModdingDocs/Images/BetterMenusKeyAndButtonBind.jpg)
## Mod Toggle
A blueprint that makes it easier to add a mod Toggle Option that takes in the ModToggleDelegates to Initialize/Unload an IToggleable Mods
```cs
MenuRef = new Menu("Example Mod Menu", new Element[]
        {
            Blueprints.CreateToggle(modToggleDelegates.Value, //the ModToggleDelegates provided by MAPI 
            "Mod Enabled", //name 
            "Should mod be enabled?"), //description
        });
```
![Mod Toggle Example](/ModdingDocs/Images/BetterMenusModToggle.jpg)
## Navigate To Menu
A blueprint to make a button to take you to a new menuscreen. Helps create more pages in the mod menu
```cs
MenuRef = new Menu("Example Mod Menu", new Element[]
        {
            Blueprints.NavigateToMenu("Extra Menu", 
                "Go to Extra Menu Page", 
                () => ExtraMenuRef.GetMenuScreen(MenuRef.menuScreen)) //this is a Func<MenuScreen> you have to return the "Next Page" MenuScreen here
        });
```
![Navigate To Menu Example](/ModdingDocs/Images/BetterMenusNavigateToMenu.jpg)
> Notice How in GetMenuScreen, MenuRef.menuScreen was passed and not modlistmenu. This is so the back button in extra menu leads to previous menu and not the `Mods` menu  
## Update Visibility
A blueprint that allows you to set visibility of multiple elements together
Makes it easier because you dont have to manually find each element and also you dont need to call MenuRef.Update (the blueprint will do it for you)
Example:
```cs
new HorizontalOption("Option Enabled", "Click here to enable/disable option",
    new string[] {"Yes", "No"},
    (index) =>
    {
        GlobalSettings.OptionIsEnabled = index == 0; //if index is equal to 0, yes was selected
        MenuRef.UpdateVisibility(GlobalSettings.OptionIsEnabled, //the value isVisible should be set to
            new []{ "MenuButton1", "KeyBind1", "MenuRow1" }); //these are a string array of element ids whose visibility needs to be updated
    },
    () => GlobalSettings.OptionIsEnabled ? 0 : 1, //if OptionIsEnabled is true, return 0 (yes) else return 1 (no)
```