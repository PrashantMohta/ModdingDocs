# ModHooks Reference
Here you can find a list of all hooks and an explanation of what they do and how to use it.

### AfterTakeDamageHook
This event is called in `HeroController.TakeDamage`. It is called after invincibility is accounted for, but before damage is applied. Can be used to change the amount of damage taken by player
> Note: Unlike the 1.4 MAPI, AfterTakeDamageHook is called when protected by baldur shell but the damage amount that is passed in is 0
```cs
ModHooks.AfterTakeDamageHook += AfterTakeDamage;
public int AfterTakeDamage(int hazardType, int damageAmount)
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
```
| Parameter type | Parameter name | Explanation                                           |
|----------------|----------------|-------------------------------------------------------|
| int            | amount         | the amount of health that will be added to the player |

| Return type | Explanation                                             |
|-------------|---------------------------------------------------------|
| int         | The new amount of health that should be added to player |


### BeforeSceneLoadHook
> Note: This hook should normally NOT be used for the following reasons:
> 1. It only allows you to change scene name and not other relevant data (like entry gate name) 
> 2. If your goal is to only see what new scene is being entered, use `UnityEngine.SceneManagement.SceneManager.activeSceneChanged` instead
> 3. If your goal is to see what scene is being entered and change it, use `On.GameManager.BeginSceneTransition` instead (because it can solve problem 1). 

This event is called in `GameManager.BeginSceneTransition`. It is called before a new scene is loaded and allows you to change what scene is loaded. However it is not recommended to use (see note above)
```cs
ModHooks.BeforeSceneLoadHook += BeforeSceneLoad;
public string BeforeSceneLoad(string newSceneName)
```
| Parameter type | Parameter name | Explanation                            |
|----------------|----------------|----------------------------------------|
| string         | newSceneName   | The name of the new scene to be loaded |

| Return type | Explanation                                     |
|-------------|-------------------------------------------------|
| string      | The new scene that the game should load instead |

### TODO
- explain hero update
- explain get/set player int/float/bool (make sure to link api docs examples)
- explain save loading hooks and why On.HC.Awake is better
- Hooks to explain
  - HeroUpdateHook
  - LanguageGetHook
  - FinishedLoadingModsHook
  - OnEnableEnemyHook (maybe)


