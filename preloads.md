# Preloads
## Introduction
To modify in game behaviors you typically need access to the GameObject that controls that behavior and to get access to gameobject, you need to be in the same "scene" or level as the gameobject. This creates a problem such that, if you want to spawn a copy of an enemy, you need access to that enemy without requiring that the player enters that scene before you can spawn it.

This is where preloads come in, preloading is a way for a mod to tell the modding api that a certain gameobject is required, the modding api then attempts to load the gameobject while the game is first loading, thus giving you access to that gameobject.

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
In the list returned by `GetPreloadNames` each element is of the form `(SceneName, GameObjectPath)` the SceneName is the internal name of a level and the GameObjectPath is the path of the game objects going from the root of the scene to the desired object.

Gameobjects can have a parent child relationship, so the GameObjectPath is the list of all ancesstors in the order of decending seniority separated by a `/`   i.e. `GreatGrandParent/GrandParent/Parent/Child` for loading Child.

The Scene name and this path allow the modding api to locate the exact object that is desired and this object will be provided to your mod during initialization. for this reason your mod must implement a `Initialize` method similar to this when expecting preloads :

```cs
public override void Initialize(Dictionary<string, Dictionary<string, GameObject>> preloadedObjects)
{
   var BossPrefab = preloadedObjects["GG_Hornet_2"]["Boss Holder/Hornet Boss 2"];
   var CardPrefab = preloadedObjects["Cliffs_01"]["Cornifer Card"];
   Object.DontDestroyOnLoad(BossPrefab);
   Object.DontDestroyOnLoad(CardPrefab);
}
``` 

you can then access your gameobject by doing `preloadedObjects[SceneName][GameObjectPath]`.

>  Note :  Preloading objects increases the over all loading time for the game to start, if at all possible, you should preload the minimum number of objects that you need. 
