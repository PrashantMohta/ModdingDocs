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
![Key and Button Bind Example](../Images/BetterMenusKeyAndButtonBind.jpg)
## Mod Toggle
```cs
MenuRef = new Menu("Example Mod Menu", new Element[]
        {
            Blueprints.CreateToggle(modToggleDelegates.Value, "Mod Enabled", "Should mod be enabled?"),
        });
```
![Mod Toggle Example](../Images/BetterMenusModToggle.jpg)
## Navigate To Menu
```cs
MenuRef = new Menu("Example Mod Menu", new Element[]
        {
            Blueprints.NavigateToMenu("Extra Menu", "Go to Extra Menu Page", () => ExtraMenuRef.GetMenuScreen(MenuRef.menuScreen))
        });
```
![Navigate To Menu Example](../Images/BetterMenusNavigateToMenu.jpg)
> Notice How in GetMenuScreen, MenuRef.menuScreen was passed and not modlistmenu. This is so the back button in extra menu leads to previous menu and not the `Mods` menu  
## Update Visibility