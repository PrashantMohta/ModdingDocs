---
title: ModHooks
nav-order: 1
parent: Hooks
---

# ModHooks
ModHooks are hooks that are built into the Modding API that allow us to interact will Hollow Knight code.
To use a Modhook, you need to subscribe to the hook. For example:
```cs
public class MyFirstMod:Mod
{
    // The method that will be called by Modding API when the game first opens
    public override void Initialize()
    {
        // We subscribe to HeroUpdateHook which is an hook that is triggered when the 'Update' Function is called for the player (once every frame)
        ModHooks.HeroUpdateHook += OnHeroUpdate;
    }
    
    //This event's method doesn't take any Parameters. normally the IDE can generate this function for you with the correct parameters
    public void OnHeroUpdate()
    {
       //code to run
    }
}
```
> Note: Your IDE (Visual Studio Community/Jetbrains Rider) can generate this function for you with the correct parameters. To do this, [see example video](https://youtu.be/oH-lbfZORw0) or type in `ModHooks.HeroUpdateHook += OnHeroUpdate;`, Then right click on the now red highlighted `OnHeroUpdate` and click on the light bulb icon (called 'Quick actions and Refactoring') and choose 'Generate Method'.

There are many modhooks available to be used.  
Here you can find a list of some hooks and an explanation of what they do and how to use it.  
However, not all hooks are listed here. Please refer to the 
[API Documentation](https://hk-modding.github.io/api/api/Modding.ModHooks.html#events) for a list of all hooks.

### HeroUpdateHook
This event is called in `HeroController.Update`.  
It can be used to run code every frame the player exists.
Useful for running code every frame during gameplay without having to create a MonoBehaviour.
Can be used to check for conditions and run code (example: check `Input.GetKeyDown(KeyCode.Space))` and run code depending on that.
```cs
ModHooks.HeroUpdateHook += HeroUpdate;
public string HeroUpdate()
{
    if Input.GetKeyDown(KeyCode.Space))
    {
        Log("Space was pressed");
    }
}
```

### LanguageGetHook
This hook is the main way to change ingame text. The way it works is that each time the game wants to find what text to display, 
it uses `Language.Language.Get` (which is where this hook is called) to get the text based on the text's "key". The text is also 
organized in sheets so the key and sheet uniquely identify the text. To see what key and sheet a text is in, refer to 
[the full list of all keys and sheets](https://docs.google.com/spreadsheets/d/1_sQ5ygsrN42toz3VnKy-8v6bwrh0UoaV/edit?usp=sharing&ouid=105022867698529839659&rtpof=true&sd=true). or add logging to the hook
To use the hook, check the `key` and `sheetTitle` and return the new text you want to display if the key/sheetTitle is the one you want to change.
Alternatively, you can use `orig` to check for text and return new text. However this is not recommended because it may cause unexpected text changes.
An example for both is given below
```cs
ModHooks.LanguageGetHook += LanguageGet;
public string LanguageGet(string key, string sheetTitle, string orig)
{
    //Check for the key and sheet for MainMenu "Yes" text
    if (key == "NAV_YES" && sheetTitle == "MainMenu")
    {
        //return the new text you want to display. 
        return "Yee";
    }
    
    //a way to replace all Yes with Yee
    if (orig.Contains("Yes"))
    {
        return orig.Replace("Yes", "Yee");
    }
    
    //make sure to return orig for all the other texts you dont want to change
    return orig;
}
```

This hook can also be used to return text for custom keys you create. (An example being using SFCore to add custom charms). To do this,
check for `if(key == "MyCustomKey") return "MyCustomText`.

| Parameter type | Parameter name | Explanation                                                                                                                           |
|----------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------|
| string         | key            | The key of the text in the sheet. This combined with sheetTitle is the way the game uniquely identifies the text required to be shown |
| string         | sheetTitle     | The sheet that the text is in                                                                                                         |
| string         | orig           | The original string. To be returned if nothing is changed                                                                             |

| Return type | Explanation                                                                   |
|-------------|-------------------------------------------------------------------------------|
| string      | The new text that should be displayed. return orig if no changes are required |

### BeforeSceneLoadHook
> Note: This hook should normally NOT be used for the following reasons:
> 1. It only allows you to change scene name and not other relevant data (like entry gate name) 
> 2. If your goal is to only see what new scene is being entered, use `UnityEngine.SceneManagement.SceneManager.activeSceneChanged` instead
> 3. If your goal is to see what scene is being entered and change it, use `On.GameManager.BeginSceneTransition` instead (because it can solve problem 1). 

This event is called in `GameManager.BeginSceneTransition`. It is called before a new scene is loaded and allows you to change what scene is loaded. However it is not recommended to use (see note above).
```cs
ModHooks.BeforeSceneLoadHook += BeforeSceneLoad;
public string BeforeSceneLoad(string newSceneName)
{
    Log($"new scene is {newSceneName}");
    return newSceneName;
}
```
| Parameter type | Parameter name | Explanation                            |
|----------------|----------------|----------------------------------------|
| string         | newSceneName   | The name of the new scene to be loaded |

| Return type | Explanation                                     |
|-------------|-------------------------------------------------|
| string      | The new scene that the game should load instead |

### SavegameLoadHook and NewGameHook
Although these 2 hooks together can provide the ability to check for save opened, it is best to avoid these two hooks because
1. NewGameHook is not called by Randomizer when creating a new game
2. Using both hooks is clunky in order to check for save opened

Instead we would recommend to use
1. `On.UIManager.StartNewGame` to check for new game load
2. `On.HeroController.Awake` to check for save opened

### FinishedLoadingModsHook
This event is called after all mod's Initialize have been called. Useful to run code that is dependant on other mods being loaded.
```cs
ModHooks.FinishedLoadingModsHook += FinishedLoadingMods;
public string FinishedLoadingMods()
{
    //checks whether the mod named DebugMod exists
    if (ModHooks.GetMod("DebugMod") is Mod)
    {
        Log("Debug Mod exists");
    }
}
```

### AfterTakeDamageHook
This event is called in `HeroController.TakeDamage`. It is called after invincibility is accounted for, but before damage is applied. Can be used to change the amount of damage taken by player
> Note: Unlike the 1.4 MAPI, AfterTakeDamageHook is called when protected by baldur shell but the damage amount that is passed in is 0
```cs
ModHooks.AfterTakeDamageHook += AfterTakeDamage;
public int AfterTakeDamage(int hazardType, int damageAmount)
{
    Log($"Damage amount is {damageAmount}");
    
    //make player take double damage
    return damageAmount * 2;
}
```
| Parameter type | Parameter name | Explanation                                                                                                                                                                      |
|----------------|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| int            | hazardType     | Even though it is an integer it represents a value from 0-4 of the enum `HazardType`. the possible values are => (0) `NON_HAZARD`,(1) `SPIKES`,(2) `ACID`,(3) `LAVA`, (4) `PIT`, |
| int            | damageAmount   | The amount of damage that was going to be applied to player. return back this value to not change damage taken                                                                   |

| Return type | Explanation                                         |
|-------------|-----------------------------------------------------|
| int         | The new damage that should be applied to the player |


### BeforeAddHealthHook
This event is called in `PlayerData.AddHealth`. It is called before normal health is added. Can be used to change the amount of health added on heal
```cs
ModHooks.BeforeAddHealthHook += BeforeAddHealth;
public int BeforeAddHealth(int amount)
{
    Log("Amount of health to be healed is {amount}");
    
    //make healing heal 2 times more health
    return amount * 2;
}
```
| Parameter type | Parameter name | Explanation                                           |
|----------------|----------------|-------------------------------------------------------|
| int            | amount         | the amount of health that will be added to the player |

| Return type | Explanation                                             |
|-------------|---------------------------------------------------------|
| int         | The new amount of health that should be added to player |

### TODO
- Explain Get/Set Player Int/Float/Bool/String/Vector3 Hooks