# Getting started with Modding Hollow Knight  

## What you will need

- A Windows / Linux / Mac device that you can code on
- A Legitimate copy of Hollow Knight on PC (Steam / GoG / HumbleBundle)
  - Console / Xbox gamepass versions do not work for our purposes.
- A copy of the [Hollow Knight Modding Api](https://github.com/hk-modding/api) binary (dll) files
  - Installing them through any modinstaller (e.g. Scarab or Butterfly) also works.
- Visual Studio Community and .NET Framework 4.7.2
  - Any version of Visual Studio or any other C# IDE works.
  - .NET Framework 4.7.2 is included in the .NET 4.8 Runtime.

## System Setup

To Setup your system for modding, you need to do the following steps. if you prefer, you can refer to the video guides below and follow along as well.

1. Download and Install [Visual Studio Community](https://visualstudio.microsoft.com/vs/community/)
1. Download and Install [.NET Framework 4.7.2](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net472).
1. Create a folder named `Hollow Knight Mods` as your working directory.
1. Skip the following 2 steps if you installed the api with a modinstaller.
1. Create a copy of your `Managed` Folder from the game files in your `Hollow Knight Mods` directory and rename it to `Vanilla` this should be unmodified copy of your original game files.
1. Create a folder named `Api` in your `Hollow Knight Mods` directory, copy the contents of the latest release of the [Hollow Knight Modding Api](https://github.com/hk-modding/api/releases) based on your operating system, this will be your copy of the modding api.

You're now done setting up your system for creating your first Hollow Knight mod.

### Windows

[![Windows guide video](https://prashantmohta.github.io/ModdingDocs/Images/step1guidewin.jpg)](https://www.youtube.com/watch?v=qT9a0k0fqqM)

### Mac

> **TODO:** Add a video here that goes over setting up and installing all of this

### Linux

[Install a .NET SDK using Microsoft's packages.](https://docs.microsoft.com/en-us/dotnet/core/install/linux)
Do note that using the .NET 6 SDK or later is preferred.
