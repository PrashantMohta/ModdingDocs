# Creating your first Mod

## Basics of C#

Before we take a deep dive into writing your first mod, we need to take a look at C#, the programming language that we would be using to create our mods, it would certainly be helpful if you know a little bit about programming already, but if not then you can use the resources in this section to get a basic understanding of programming and C#.

 - [Csharp 101 Video Series](https://docs.microsoft.com/en-us/shows/csharp-101/)
 - [A tour of the C# language Article Series](https://docs.microsoft.com/en-us/dotnet/csharp/tour-of-csharp)

The video series is especially beginner friendly, so you may want to try that !

## Basic structure of a mod

let's take a look at a very basic mod and break it down, this is a mod that logs some text to our `modlog.txt` file every frame the player presses `O` on their keyboard.

```cs 
using System;
using System.Collections;
using System.Collections.Generic;
using Modding;
using UnityEngine;

namespace MyFirstMod
{
    public class MyFirstMod: Mod 
    {
        new public string GetName() => "My First Mod";
        public override string GetVersion() => "v1";
        public override void Initialize()
        {
            ModHooks.HeroUpdateHook += OnHeroUpdate;
        }
        public void OnHeroUpdate()
        {
           if(Input.GetKeyDown(KeyCode.O))
           {
               Log("Key Pressed")
           }
        }
    }
}
```
so what is going on here ?

 - The first few lines are `using` statements, where we tell the C# compiler the namespaces that we are going to be using in our file.
 - Next, we create our own `namespace` called `MyFirstMod`  this will contain our mod
 -  `public class MyFirstMod : Mod`  we define a new public class called `MyFirstMod` that extends the `Mod` base class, this is the class that will be loaded by the Modding api, and this is where we will tell the game about our mod.
 - `new public string GetName() => "My First Mod";` The Mod class has a few special methods that if provided by out mod allows us to do a few interesting things. `GetName()` for example allows us to set our mods name as it will appear in game, in the top-left mods list.
 -  `GetVersion` serves a similar purpose but for the version of our mod.
 -  `Initialize` is called when the Api is ready for our mod to start making modifications, this is also the place where preloads are provided (more on that in a bit)
 - `ModHooks.HeroUpdateHook` the Modding api provides us with certain *events* that we can react to, this is what allows us to code things that respond to in game events. `HeroUpdateHook` for example is an event that is invoked every frame that the player character exists, this makes it possible for us to listen for a keypress and react to that, for achieving this we use our own method `OnHeroUpdate`.
 - `OnHeroUpdate` simply checks if the `O` key was pressed down this frame, if it was then it adds the string "Key Pressed" to the `Modlog.txt`

## The Mod base Class & Mod Lifecycle 
For further reference into what the mod base class is and the lifecycle of a mod you can visit these reference docs. 
 - [The Mod Class](mod-baseclass.md)
 - [The Mod Lifecycle](mod-lifecycle.md) 

## Installing the Mod Template

Strictly speaking, there is no need to use a template to create your mod. it will however take some common things that you will have to do every time you start a project and do them for you, which is nice.

To install the template : 
 - Download the [Template Zip](https://cdn.discordapp.com/attachments/879130756146954240/931586813729075270/Hollow_Knight_1.5_Mod.zip) 
 - Move this zip to `{documents folder}\Visual Studio 2022\Templates\ProjectTemplates`
 - Restart visual studio 

## Create your Mod using the Mod Template

> TODO add steps here 

 - Fix the assembly references in your mod project
 - Add the Code from the mod in [Basic structure of a mod](#basic-structure-of-a-mod) section in a new file named `MyFirstMod.cs`
 - Create a build to compile your first mod

## Time to load your mod into game

The Mod Template automatically copies the compiled `dll` of your mod into the right folder when a successful build is created (and the game is not running), so all you have to do next is start the game and you should see your mod in the top - left.

if you start a save and press `O` your `modlog.txt` will now contain the text `Key Pressed`. you can find your modlog by following the steps outlined [here](logging.md#finding-your-modlog)

>TODO add instructions to enable modlog console

