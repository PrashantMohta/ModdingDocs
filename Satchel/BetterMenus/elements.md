---
title: BetterMenus Elements
nav_order: 1
parent: BetterMenus
grand_parent: Satchel
---
# BetterMenus Elements

## 1. Horizontal Option
The basic menu option that allows you to let the user select 1 option from a list of options.

#### Example when you want to deal with bools (True/False)
```cs
 new HorizontalOption(
 		name: "Feature Active", 
        description: "Should this feature be active?",
		values: new [] { "Yes", "No" },  
		applySetting: index =>
		{
			GlobalSettings.featureActive = index == 0; //"yes" is the 0th index in the values array
        },
        loadSetting: () => GlobalSettings.featureActive ? 0 : 1), //return 0 ("Yes") if active and 1 ("No") if false
```

#### Example when you want to deal with integers:
```cs
new HorizontalOption(
        name: "Health",
        description: "How much health should this boss have",
        values: new[] { 100, 500, 1000 },
        applySetting: index =>
        {
            GlobalSetiings.health = index switch
            {
                0 => 100,
                1 => 500,
                2 => 1000,
                _ => 0 //if its has unexpected value, set it to the 0th index (100 health)
            };
        },
        loadSetting: () => GlobalSettings.health switch
        {
            100 => 0,
            500 => 1,
            1000 => 2,
            _ => 0 //if global setting has unexpected value, set it to the 0th index (100 health)
        }
    ),
```

#### Example when you want to deal with enums:
```cs
//example defination of enum
enum Modes
{
    Mode1,
    Mode2,
}
Modes chosenMode = Modes.Mode1;

//example implementation (for enums that follow the default values 0,1,2,...n)
new HorizontalOption(
        name: "Choose Mode", 
        description: "Choose from the following modes",
        values: Enum.GetNames(typeof(Modes)), //get an string array of all values in Modes enum
        ApplySetting: (index) => //the function that will be run when a new option is set
        { 
            chosenMode = (Modes)index; //convert integer to enum
        },
        loadSetting: () => (int)chosenMode), //convert enum to integer

```

#### Example when you want to store an integer but want to display a string:
```cs
new HorizontalOption(
        name: "Duration",
        description: "The Duration that this event will last",
        values: new[]{ "15s", "30s", "1min", "2min", "5min", "10min" },
        applySetting: index =>
        {
            GlobalSetiings.duration = index switch
            {
                0 => 15,
                1 => 30,
                2 => 60,
                3 => 120,
                4 => 300,
                5 => 600,
            };
        },
        loadSetting: () => GlobalSettings.duration switch
        {
            10 => 0,
            30 => 1,
            60 => 2,
            120 => 3,
            300 => 4,
            600 => 5,
            _ => 0 //if global setting has unexpected value, set it to the 0th index (15 seconds)
        }
    ),
```
See also:
- [ModToggle Blueprint](blueprints.md#mod-toggle)


## 2. Menu Button
#### Allows you to execute code when user presses this. 
Example:
```cs
new MenuButton(
        name: "My Button", //the name of the button
        description: "Does something when pressed", //the description of the button
        submitAction: (Mbutton) => 
        {
        	//execute code here when the user presses this button. 
        	//you can use the Mbutton parameter to edit the menu button if you wish
        	Log("My Button was pressed");
        }
        Id: "mybutton")// the id of the button to search for it
```
See also:
- [Navigate To Menu Blueprint](blueprints.md#navigate-to-menu)

## 3. Text Panel
Allows you to place text on the menu screen
Example using default parameters: 
```cs
new TextPanel(name:"This page was made by Mulhima"), //the text to be shown
```
Example using optional parameters: 
```cs
new TextPanel(
	name:"This page was made by Mulhima", //the text to be shown
	fontSize: 50), //the font size of the text
```
## 4. Key Bind
Gives user ability to bind a keyboard key to a custom `PlayerAction`.  
Make sure to use the method outlined in [settings page](../../saving-mod-data.md#keybinds) to save it in global settings.
Example:
```cs
new KeyBind(
    name: "ExampleKey", 
    playerAction: GlobalSettings.keyBinds.ExampleKey),
```
See also:
- [Key And Button Bind Blueprint](blueprints.md#key-and-button-bind)
## 5. Button Bind
Gives user ability to bind a controller button to a custom `PlayerAction`
Make sure to use the method outlined in [settings page](../../saving-mod-data.md#keybinds) to save it in global settings.
Example:
```cs
new ButtonBind(
    name: "ExampleButton", 
    playerAction: GlobalSettings.keyBinds.ExampleButton),
```
See also:
- [Key And Button Bind Blueprint](blueprints.md#key-and-button-bind)
## 6. Custom Slider
Allows user to select a number within a range of integers,
Example:
```cs
new CustomSlider(
   	name: "Damage Taken",
    storeValue: val => // to store the value when the slider is changed by user
    {
        GlobalSettings.damageTaken = (int) val;
    },
    loadValue: () => GlobalSettings.damageTaken, //to load the value on menu creation
    minValue: 0,
    maxValue: 10,
    wholeNumbers: true,
    ),
```
## 7. Static Panel
An empty element in the menu, can be used to add whatever you want to it