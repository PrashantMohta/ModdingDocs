# Getting started with Modding Hollow Knight

> Please note that this documentation is being written with the latest patch of the game in mind, for older patches you can refer to the documentation at [radiance.host](https://radiance.host/apidocs/Getting-Started.html)

## Introduction
Hollow Knight ( patch 1.5.x.x onwards ) is a game built with Unity 2020.2.2 and  C# ( .NET Framework 4.7.2 ) , This means that a mod in the context of hollow knight is a `dll` or a Dynamic-link library that allows players to modify the behavior of the game, fix issues or add new features. This is achieved with the help of a Modding Api that handles loading the mods and giving us ways of injecting our code in the middle of normal logic.

The [Hollow Knight Modding Api](https://github.com/hk-modding/api)   is based on [MonoMod](https://github.com/MonoMod/MonoMod), a General purpose .NET assembly modding "basework".

## What you will need

 - A Windows / Linux / Mac device that you can code on.
 - A Legitimate copy of Hollow Knight on PC (Steam / GoG / HumbleBundle).
 *Console / Xbox gamepass versions do not work for our purposes.*
 - A copy of the [Hollow Knight Modding Api](https://github.com/hk-modding/api) binary (dll) files
 - Visual Studio Community and .NET Framework 4.7.2 

## System Setup 

To Setup your system for modding, you need to do the following steps. if you prefer, you can refer to the video guides below and follow along as well.

 1. Download and Install [Visual Studio Community](https://visualstudio.microsoft.com/vs/community/) 
 2. Download and Install [.NET Framework 4.7.2](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net472)
 3. Create a folder named `Hollow Knight Mods` as your working directory.
 4. Create a copy of your `Managed` Folder from the game files in your `Hollow Knight Mods` directory and rename it to `Vanilla` this should be unmodified copy of your original game files.
 5. Create a folder named `Api` in your `Hollow Knight Mods` directory, copy the contents of the latest release of the [Hollow Knight Modding Api](https://github.com/hk-modding/api/releases) based on your operating system, this will be your copy of the modding api.

You're now done setting up your system for creating your first hollow knight mod.

### Mac 
> TODO Add a video here that goes over setting up and installing all of this
### Windows 
> TODO Add a video here that goes over setting up and installing all of this
### Linux 
> TODO Add a video here that goes over setting up and installing all of this

