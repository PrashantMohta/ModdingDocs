---
title: BetterMenus Extras
parent: BetterMenus
grand_parent: Dependency Mods
---
# BetterMenus Extras

## Notes on building pages
- You should create a new MenuRef for each menuscreen you want.
- Each "Page" needs it own menuscreen

The basic code structure can look like this
```cs
public MenuScreen GetMenuScreen(MenuScreen modListMenu, ModToggleDelegates? toggleDelegates)
{
    ExtraMenuRef ??= new Menu("My Extra Menu", new Element[]
    {
         //add elements here
    });
    
    MenuRef ??= new Menu("My Mod Menu", new Element[]
    {
         //add elements here
    });
    
    MenuRef.GetMenuScreen(modListMenu);
    ExtraMenuRef.GetMenuScreen(MenuRef.menuScreen);
    
    return MenuRef.menuScreen;
}
```

## Options To Add In The Menu Other Than The 8 Elements
Are you unhappy with the current options satchel provides in better menus? If so read bellow for some options you have to help solve it

1) if you want to simplify making an existing menu element because some of the parameters are fixed or very similar, make a function that returns an element similar to a [blueprint](blueprints.md).  
   [Example 1](https://github.com/PrashantMohta/Satchel/blob/542960d0af1361565abfd54ec50f8ffc42167b53/BetterMenus/Blueprints/NavigateToMenu.cs#L4-L12)  
   [Example 2](https://github.com/TheMulhima/Satchel/blob/52c396962a02f4aaff91c59d9f7623229a5ff973/BetterMenus/Blueprints/IntOption.cs#L33-L38)  

2) If you want to add something to an element that's not available as a parameter, use your menuref's OnBuilt event to do it.
   [Example 1](https://github.com/TheMulhima/HollowKnight.RandomTeleport/blob/96b65a9fed133ece79f2d30bcd9964e0e9a24e60/RandomTeleport/Settings/ModMenu.cs#L204-L208)

3) if you want a custom element that doesn't need any special handling in Update or doesn't need the `Content Area` from ICustomMenuMod, create a static panel and use its `CreateCustomItem` parameter to create your element (It's an Action<GameObject> which will provide you with the gameobject in the menu to which you can add components and stuff)

4) if your element is nothing like anything in satchel, Create a new class and inherit from the abstract Element class to create a new element and add that custom element to the menu like you'd do normally.  
   [Element base class](https://github.com/PrashantMohta/Satchel/blob/542960d0af1361565abfd54ec50f8ffc42167b53/BetterMenus/Base/Element.cs#L69-L84)  
   [Example 1](https://github.com/PrashantMohta/Satchel/blob/542960d0af1361565abfd54ec50f8ffc42167b53/BetterMenus/Elements/CustomSlider.cs#L6-L165)  
   If you do make a custom element, a PR to [Satchel](https://github.com/PrashantMohta/Satchel) would be appreciated as it would benefit everyone who uses Satchel

## Adding Elements in ForLoops
If you have many very similar elements to build or an abstraction that only returns Elements, you can still very easily create the Menu. The basic code would look like this
```cs
public MenuScreen GetMenuScreen(MenuScreen modListMenu, ModToggleDelegates? toggleDelegates)
{
    MenuRef ??= new Menu("My Mod Menu", Array.Empty<Element>); //create a new menu
    for (int i = 0; i < 5; i++)
    {
      MenuRef.AddElement(new TextPanel($"Option {i}", fontSize: 55)); //add a text panel
      HKMPMenu.AddElement(new KeyBind($"Keybind {i}", settings.Keybinds.keysList[i])); //add a keybind
      HKMPMenu.AddElement(new StaticPanel("Empty Space", _ => { })); //add a empty static panel
    }
    return MenuRef.GetMenuScreen(modListMenu);
}
```
## Events
Events help run code when certain actions happen in the Menu. The available events are:
1. OnBuilt - When menu is built. This happens once when game opens and everytime UIManager is destroyed (when save is closed).
2. OnUpdate -
4. OnVisibilityChange - When visibility of an element changes (when Element.Show() or Element.Hide() is used)
5. OnReflow - When menu is reflowed to account for changed to the visibility of elements in the menu

Events can be subscribed to anywhere but its best to subscribe to them after MenuRef is created but before MenuScreen is returned in the GetMenuScreen function.
For the parmeters of the events, the first parameter can be safely discarded while the second parameter is element that is the target. It can be a `Menu` or a `Element`.
) Note: Be careful to check for if MenuRef == null before subscribing to events. Otherwise it might lead to re-subscribing
