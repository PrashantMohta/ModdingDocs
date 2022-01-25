# Hooks
Hooks (more formally known as events) are the main way to interact with Hollow Knight's code. We can use them to wait for an event to happen in game, then intercept data related to that event and trigger some code and potentially return different data to what we received.   
<br>There are 3 major type of hooks in the Modding API each of which will be explained below.

## ModHooks
ModHooks are hooks that are built into the Modding API that allow us to interact will Hollow Knight code.
To use a modhook, you need to subscribe to the hook. For example:
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
> Note: Your IDE (Visual Studio Community/Jetbrains Rider) can generate this function for you with the correct parameters. To do this, type in `ModHooks.HeroUpdateHook += OnHeroUpdate;`, Then right click on the now red highlighted `OnHeroUpdate` and click on the light bulb icon (called 'Quick actions and Refactoring') and choose 'Generate Method'.

There are many modhooks available to be used.  
- Explanations for most common hooks can be found in [ModHook Reference](Hooks/modhooks.md)
- A list of all hooks can be found in the [API Documentation](https://hk-modding.github.io/api/api/Modding.ModHooks.html#events)
## On Hooks
### TODO On Hooks
- Add explanation of on Hooks
- explain orig(self)
- Link to on hooks page
## IL Hooks
### TODO IL Hooks
- simple explanation of what they are and what is IL
- when to use
- link to IL hook page

### Todo General
-  how use custom values from player data & language (@dandy)