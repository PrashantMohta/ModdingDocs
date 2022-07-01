---
nav_order: 2
---
# Environment setup  

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

1. Download and Install [Visual Studio Community](https://visualstudio.microsoft.com/vs/community/).
    - Make sure to also install the "C# for desktop development" workflow.
2. Download and Install [.NET Framework 4.7.2](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net472).
    - For linux see [Install a .NET SDK using Microsoft's packages.](https://docs.microsoft.com/en-us/dotnet/core/install/linux) 
      Do note that using the .NET 6 SDK or later is preferred.
3. Create a folder named `Hollow Knight Mods` as your working directory (you will create your mods here).
4. Install the Modding Api (skip if already using a modded install of the game.)
    - Either by using a modinstaller (auto installs on load)
    - Or manually by replacing the files in your `Managed` Folder in the game files with the contents of [Hollow Knight Modding Api](https://github.com/hk-modding/api/releases)'s latest release for your operating system, this will be your copy of the modding api.

You're now done setting up your system for creating [your first Hollow Knight mod](/your-first-mod.md).

### Windows

[![Windows guide video](/ModdingDocs/Images/step1guidewin.jpg)](https://www.youtube.com/watch?v=qT9a0k0fqqM)

### Mac

> **TODO:** Add a video here that goes over setting up and installing all of this

### Linux


> **TODO:** Add a video here that goes over setting up and installing all of this
