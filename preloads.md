---
nav_order: 6
---
# Preloading game objects
## Introduction
To modify in game behaviors you typically need access to the GameObject that controls that behavior. To get access to said GameObject, you need to be in the same "scene" or level as it. This creates a problem such that, if you want to spawn a copy of an enemy, you need access to that enemy without needing the player to be in that scene before you can spawn it.

This is where preloads come in, preloading is a way for a mod to tell the modding api that a certain gameobject is required. The modding api then attempts to load the GameObject while the game is first loading, thus giving you access to that gameobject.

## How to preload an object

To preload a gameobject your mod class must implement a method `GetPreloadNames` like so :

```cs
public override List<(string, string)> GetPreloadNames()
{
    return new List<(string, string)>
    {
        ("GG_Hornet_2", "Boss Holder/Hornet Boss 2"),
        ("Cliffs_01","Cornifer Card")
    };   
}
```
In the list returned by `GetPreloadNames` each element is of the form `(SceneName, GameObjectPath)`. `SceneName` is the internal name of a level and `GameObjectPath` is the path of the GameObjects going from the root of the scene to the desired object.

GameObjects can have a parent child relationship, so the GameObjectPath should be a list of all ancestors in the order of decending seniority separated by a `/`.  I.e. `GreatGrandParent/GrandParent/Parent/Child` for loading Child.

> Note: you can use the [Scene Dump](https://prashantmohta.github.io/ModdingDocs/#todo-section) to find the path of a particular GameObject.

The SceneName and this path allow the modding api to locate the exact object that is desired and this object will be provided during initialization. For this reason your mod must implement a `Initialize` method similar to this when expecting preloads:

```cs
public override void Initialize(Dictionary<string, Dictionary<string, GameObject>> preloadedObjects)
{
   var BossPrefab = preloadedObjects["GG_Hornet_2"]["Boss Holder/Hornet Boss 2"];
   var CardPrefab = preloadedObjects["Cliffs_01"]["Cornifer Card"];
   Object.DontDestroyOnLoad(BossPrefab);
   Object.DontDestroyOnLoad(CardPrefab);
}
``` 
You can then access your GameObject by doing `preloadedObjects[SceneName][GameObjectPath]`.

> Note: Preloading objects increases the overall loading time for the game to start. If possible, you should preload the minimum number of objects that you need. 
