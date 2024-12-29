---
nav_order: 3
---
# Creating your first Mod

> Note : if you prefer to follow along with a video see [the modding tutorial](#video-tutorial)

## Basics of C#

Before we take a deep dive into writing your first mod, we need to take a look at C#, the programming language that we would be using to create our mods, it would certainly be helpful if you know a little bit about programming already, but if not then you can use the resources in this section to get a basic understanding of programming and C#.

- [Csharp 101 Video Series](https://docs.microsoft.com/en-us/shows/csharp-101/)
- [A tour of the C# language Article Series](https://docs.microsoft.com/en-us/dotnet/csharp/tour-of-csharp)

The video series is especially beginner friendly, so you may want to try that !

## Unity Fundamentals

While not entirely mandatory, knowing about unity concepts is immensely helpful in creating the mental model required for making your own mods. [This video by Game Maker's Toolkit](https://www.youtube.com/watch?v=XtQMytORBmM) is a good introduction to unity. I highly recommend watching it.

## Installing the Mod Template

> Tip: if your IDE supports it, using an [extension](ide-extensions.md) can simplify or eliminate many of the
steps below!

Strictly speaking, there is no need to use a template to create your mod. it will however take some common things that you will have to do every time you start a project and do them for you, which is nice.

To install the template :
- Download the [Template Zip](https://github.com/PrashantMohta/ModdingDocs/releases/download/1.5-modding-template/Hollow_Knight_1.5_Mod.zip)
- Move this zip to `{documents folder}\Visual Studio 2022\Templates\ProjectTemplates`
- Restart visual studio

## Create your Mod using the Mod Template

- Open visual studio community 2022
- Click "Create a new project" on the right
- Select the Hollow Knight 1.5 Mod template
- Enter project name and change the location to your working directory
- [optional] Select place solution and project in the same directory
- Fix the assembly references in your mod project
    > Note : this can be done easily by opening the csproj file in your project directory
- Add the Code from the mod in [Basic structure of a mod](#basic-structure-of-a-mod) section in a new file named `MyFirstMod.cs`
- Create a build to compile your first mod

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
    public class MyFirstMod : Mod
    {
        public MyFirstMod() : base("My First Mod") { }
        public override string GetVersion() => "v1";
        public override void Initialize()
        {
            ModHooks.HeroUpdateHook += OnHeroUpdate;
        }
        public void OnHeroUpdate()
        {
           if(Input.GetKeyDown(KeyCode.O))
           {
               Log("Key Pressed");
           }
        }
    }
}
```
so what is going on here ?

- The first few lines are `using` statements, where we tell the C# compiler the namespaces that we are going to be using in our file.
- Next, we create our own `namespace` called `MyFirstMod`  this will contain our mod
- `public class MyFirstMod : Mod`  we define a new public class called `MyFirstMod` that extends the `Mod` base class, this is the class that will be loaded by the Modding api, and this is where we will tell the game about our mod.
- `public MyFirstMod() : base("My First Mod") { }` The constructor of the Mod class has a parameter that allows us to set the mods name as it will appear in game, in the top-left mods list.
- `GetVersion` The Mod class has a few special methods that if provided by out mod allows us to do a few interesting things. `GetVersion()` for example allows us to set the version of our mod that will be displayed alongside the name in the top-left mods list.
- `Initialize` is called when the Api is ready for our mod to start making modifications, this is also the place where preloads are provided (more on that in a bit)
- `ModHooks.HeroUpdateHook` the Modding api provides us with certain *events* that we can react to, this is what allows us to code things that respond to in game events. `HeroUpdateHook` for example is an event that is invoked every frame that the player character exists, this makes it possible for us to listen for a keypress and react to that, for achieving this we use our own method `OnHeroUpdate`.
- `OnHeroUpdate` simply checks if the `O` key was pressed down this frame, if it was then it adds the string "Key Pressed" to the `Modlog.txt`

## The Mod base Class & Mod Lifecycle

For further reference into what the mod base class is and the lifecycle of a mod you can visit these reference docs.
- [The Mod Class](mod-baseclass.md)
- [The Mod Lifecycle](mod-lifecycle.md)

## Time to load your mod into game

The Mod Template automatically copies the compiled `dll` of your mod into the right folder when a successful build is created (and the game is not running), so all you have to do next is start the game and you should see your mod in the top - left.

if you start a save and press `O` your `modlog.txt` will now contain the text `Key Pressed`. you can find your modlog by following the steps outlined [here](logging.md#finding-your-modlog)


## Video tutorial

you can follow along with this video tutorial to create your first mod.  
[![creating your first mod](/ModdingDocs/Images/firstmod.jpg)](https://youtu.be/LMayapYyEr8)
