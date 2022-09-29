---
title: Blueprints
nav_order: 2
parent: BetterMenus
grand_parent: Satchel
---
# BluePrints
Blueprints is a class that has been created to help provide and ease the creation of commonly used functionality in modmenus.

## Confirm Dialogue
Add a new screen that shows up when back button or escape is pressed that acts as a confirm dialogue
Example:
```cs
public MenuScreen GetMenuScreen(MenuScreen modListMenu, ModToggleDelegates? toggleDelegates)
{
    //create your menu screen
    MenuRef ??= new Menu("Breakable Charms", new Element[]
    {
        //add elements here
    });

    // we need to create a menuscreen before a confirm dialogue can be attached
    var menuScreen = MenuRef.GetMenuScreen(modlistmenu);

    //after creation we can call the function
    MenuRef.AddConfirmDialog(
        title: "Are you sure",
        subTitle: "Do you really wanna go back",
        Options: new string[] { "Yes", "No" }, //the list of buttons that will be shown
        OnButtonPress: (selection) => //what will happen on any of the button press. 
        //Note that selection is a string and we have to check which button is pressed
        //by equating the button names (as given above) :(
        {
            switch (selection)
            {
                case "Yes": //if yes, take user to previous menu (modlist menu in this case)
                    Satchel.BetterMenus.Utils.GoToMenuScreen(modlistmenu);
                    break;
                case "No": //if no, take user back to inital menu (the one that this is attached to)
                    Satchel.BetterMenus.Utils.GoToMenuScreen(MenuRef.menuScreen);
                    break;
            }
        });
    return menuScreen; //return our menuscreen to mapi
}
```
![Confirm Dialogue Example](/ModdingDocs/Images/BetterMenusConfirmDialogue.jpg)

Notes:
- Unfortunately the order given must be followed (MenuRef creation -> calling MenuRef.GetMenuScreen -> creating confirm dialogue -> returning menu screen)
- if you want the confirm dialogue to contain more buttons, you can add those to the Options array just as the example. Those buttons will be placed 2 per row unless specified using the optionsPerRow optional parameter
- if you want your confirm dialogue to have anything else or want more customization you can use ` MenuRef.AddConfirmDialog` which takes in a MenuRef as parameter. In that you can add what you want
- If user presses esc, they will be taken back to inital menu (the one that the confirm dialogue is attached to)


## Key And Button Bind
A blueprint to create a row with keybind and a button bind where only the binding area of them are selectable
```cs
MenuRef = new Menu("Example Mod Menu", new Element[]
        {
            Blueprints.KeyAndButtonBind(
                name: "Action 1",
                keyBindAction: keyBinds.Key1, 
                buttonBindAction: keyBinds.Button1),
        });
```
![Key and Button Bind Example](/ModdingDocs/Images/BetterMenusKeyAndButtonBind.jpg)
## Mod Toggle
A blueprint that makes it easier to add a mod Toggle Option that takes in the ModToggleDelegates to Initialize/Unload an IToggleable Mods
```cs
MenuRef = new Menu("Example Mod Menu", new Element[]
        {
            Blueprints.CreateToggle(
                toggleDelegates: modToggleDelegates.Value, //the ModToggleDelegates provided by MAPI 
                name: "Mod Enabled",  
                description: "Should mod be enabled?"),
        });
```
![Mod Toggle Example](/ModdingDocs/Images/BetterMenusModToggle.jpg)
## Navigate To Menu
A blueprint to make a button to take you to a new menuscreen. Helps create more pages in the mod menu
```cs
MenuRef = new Menu("Example Mod Menu", new Element[]
        {
            Blueprints.NavigateToMenu(
                name: "Extra Menu", 
                description: "Go to Extra Menu Page", 
                getScreen: () => ExtraMenuRef.GetMenuScreen(MenuRef.menuScreen)) //this is a Func<MenuScreen> you have to return the "Next Page" MenuScreen here
        });
```
![Navigate To Menu Example](/ModdingDocs/Images/BetterMenusNavigateToMenu.jpg)
> Notice How in GetMenuScreen, MenuRef.menuScreen was passed and not modlistmenu. This is so the back button in extra menu leads to previous menu and not the `Mods` menu  
## Update Visibility
A blueprint that allows you to set visibility of multiple elements together
Makes it easier because you dont have to manually find each element and also you dont need to call MenuRef.Update (the blueprint will do it for you)
Example:
```cs
new HorizontalOption(
        name: "Option Enabled",
        description: "Click here to enable/disable option",
        values: new string[] {"Yes", "No"},
        applySetting: (index) =>
        {
            GlobalSettings.OptionIsEnabled = index == 0; //if index is equal to 0, yes was selected
            MenuRef.UpdateVisibility(GlobalSettings.OptionIsEnabled, //the value isVisible should be set to
                new []{ "MenuButton1", "KeyBind1", "MenuRow1" }); //these are a string array of element ids whose visibility needs to be updated
        },
        loadSetting: () => GlobalSettings.OptionIsEnabled ? 0 : 1) //if OptionIsEnabled is true, return 0 (yes) else return 1 (no)
```
## Get Cached MenuScreen
A blueprint that gives you back an old menuscreen if it is available. Best used in NavigateToMenu Blueprint
Example:
```cs
Blueprints.NavigateToMenu(
        name: "Extra Menu", 
        description: "Go to Extra Menu Page", 
        getScreen: () => ExtraMenuRef.GetCachedMenuScreen(MenuRef.menuScreen)),
```