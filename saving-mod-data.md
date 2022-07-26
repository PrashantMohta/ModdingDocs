---
nav_order: 9
---
# Saving Mod Data

## Introduction 
There are many times where you want data created by your mod to persist after the game closes. The primary way to to this via the MAPI is thorough the LocalSettings and GlobalSettings interfaces. A LocalSetting is a setting that is specific to a save file while a GlobalSetting is a setting that is common between all saves. These interfaces allow you to store data as a JSON in the saves folder which can be read with old data when your mod is loaded and written to with new data when game is closed. Since it's stored as a JSON, a user can edit the values in the file which allows you to let the user configure some settings if they wish. 

> Note: The 1.5 MAPI provides a ModMenu interface which allows users to change settings in-game which is preferred, as compared to asking users to find a specific file in the save folder to change settings.

A mod's global settings can be found in the saves folder with the name `ModName.GlobalSettings.json` while it's local settings are found in `userN.modded.json` where N is the save file number

## How to use
It is really easy to create a settings interface. For this explanation global settings will be used and an example for local settings will be provided under that.  
The first step is to create a class and in that class add fields that will store your persistent data/configurations

Example
```cs
public class GlobalSettingsClass
{
    public bool shouldRun = true; //create a variable that stores whether the mod should run or not. Default value (when mod first installed) is true
    public int wins = 5; //default value is 5
    public string characterName = "Knight";
    public float bonusDamage = 2.5f;
    public List<string> ScenesVisited = new List<string>(); //default value is empty list
}
```

These are just some examples but you can use any field type that is serializeable (bool, int, float, double, string, List, Dictionary, etc.). However not all fields are serializeable and the ways around it are listed in the the [Notes on Non Serializeable Fields Section](#notes-on-non-serializeable-fields)

The next step is to tell MAPI that you want a global setting. You do this by making the class that inherits from `Mod` to also inherit from the interaface `IGlobalSettings<T>` where T is the settings class name.  
This interface requires you to implement 2 methods `OnLoadGlobal`(what happens when global settings is read from file) and `OnSaveGlobal` (what happens when global settings is saved to file). In most cases you want to use OnLoadLocal to save the data read in a local variable and OnSaveLocal to return that local variable

Example:
```cs
public class MyFirstMod: Mod, IGlobalSettings<GlobalSettingsClass>
{
    //Create and initalize a local variable to be able to access the settings
    public static GlobalSettingsClass GS {get; set;} = new GlobalSettingsClass();

    // First method to implement. The parameter is the read settings from the file
    public void OnLoadGlobal(GlobalSettingsClass s)
    {
        GS = s; // save the read data into local variable;
    }

    public GlobalSettingsClass OnSaveGlobal()
    {
    	return GS;//return the local variable so it can be written in the json file
    }

}
```

Now with this, the global settings is now ready to use. To access a value you can do it through the GS variable (example: `GS.shouldRun`)

Example of implementing Local Settings:
```cs
public class MyFirstMod: Mod, ILocallSettings<LocalSettingsClass>
{
	public static LocalSettingsClass saveSettings { get; set; } = new LocalSettingsClass();
	public void OnLoadLocal(LocalSettingsClass s) => saveSettings = s;
    public LocalSettingsClass OnSaveLocal() => saveSettings;
```


## Notes on Non Serializeable Fields
An indicator for a type not being seralizable is that it doesnt show up in the JSON file. Normally any public field or property with no non-public accessors is serialized, by default. For more information you can check the [Newtonsoft documentation](https://www.newtonsoft.com/json/help/html/Introduction.htm)

### Enums
To make sure the enums are stored as strings and not ints in the settings file, you can add the attribute `[JsonConverter(typeof(StringEnumConvertor))]` to the field. Even if you dont add the attribute it will still seralize but with the attribute, it will make it easier for people to edit.

```cs
public enum CurrentState
{
    First = 0,
    Second
}

public class GlobalSettingsClass
{
    [JsonConverter(typeof(StringEnumConvertor))]
    public CurrentState state = CurrentState.First;
}
```

### Keybinds
When creating a mod menu, there is an option to have bindable keys. However, to save those keys in a settings file, we need to use this method.  

The first step is to create a Keybinds Class that inherts from `PlayerActionSet`
```cs
public class KeyBinds : PlayerActionSet
{
	//the keybinds you want to save. it needs to be of type PlayerAction
    public PlayerAction Key1;

    //a constructor to initalize the PlayerAction
    public KeyBinds()
    {
        Key1 = CreatePlayerAction("Key1");

       //optional: set a default bind
        Key1.AddDefaultBinding(Key.A);
    }
```

The next step is to add a field of type `KeyBinds` in the settings class and add the `[JsonConverter(typeof(PlayerActionSetConverter))]` attribute to that field
```cs
public class GlobalSettingsClass
{
	[JsonConverter(typeof(PlayerActionSetConverter))]
	public KeyBinds keybinds = new KeyBinds();
}
```

Now you can access the keybinds by doing `GS.keybinds.Key1`

### Vector2 and Vector3
The MAPI provides a way to seralize those. to do this add the `JsonConverter` attribute to the vector2/vector3 field
```cs
public class GlobalSettingsClass
{
     [JsonConverter(typeof(Vector2Converter))]
     public Vector2 myVector2 = new Vector2(0,0);


     [JsonConverter(typeof(Vector3Converter))]
     public Vector3 myVector3 = new Vector3(0,0,0);
}
```

### Using workarounds
If the type you are trying to save is not seralizable, you can use work arounds to save it in global settings. For example: if you wanted to save a `UnityEngine.Color`, you can do
```cs
public class GlobalSettingsClass
{
	public int IconColorR = 0;
	public int IconColorG = 0;
	public int IconColorB = 0;

	//adding a JsonIgnore attribute to the property will make the seralizer ignore the property
    [JsonIgnore]
    public Color IconColor
    {
        get => new Color(IconColorR, IconColorG, IconColorB);
        set
        {
            IconColorR = Mathf.RoundToInt(value.r); 
            IconColorG = Mathf.RoundToInt(value.g); 
            IconColorB = Mathf.RoundToInt(value.b); 
            
        }
    }
}
```

### Creating your own custom JSONConvertors
If you don't want to do a workaround, you can create your own JSON Convertor. Examples are linked below:
1. [System.Random](https://github.com/TheMulhima/HollowKnight.RandomTeleport/blob/master/RandomTeleport/JsonConvertors/RandomJsonConvertor.cs)
2. [Vector2](https://github.com/hk-modding/api/blob/master/Assembly-CSharp/Converters/Vector2Converter.cs)
3. [Vector3](https://github.com/hk-modding/api/blob/master/Assembly-CSharp/Converters/Vector3Converter.cs)
4. [PlayerActionSet](https://github.com/hk-modding/api/blob/master/Assembly-CSharp/Converters/PlayerActionSetConverter.cs)
