# Hollow Knight Mod Templates (NuGet)

This is a NuGet package for making new hollow knight mods. [See it on the NuGet Gallery](https://www.nuget.org/packages/HKModding.HKMod.Templates/1.0.14)

**Keep in mind using the NuGet package instead of the IDE Template can often be more difficult.**
If you are looking for the IDE Template, [this](https://prashantmohta.github.io/ModdingDocs/your-first-mod.html#installing-the-mod-template) might be it.

## Installation
### From NuGet gallery (recommended)
1.  From a terminal, run `dotnet new -i HKModding.HKMod.Templates`.
### Manual installation
1.  Go to the Releases page (https://github.com/jngo102/HKModTemplates/releases) and download the latest `HKModTemplates.nupkg` file (not the Source Code!). 
2.  From a terminal, run `dotnet new -i {/path/to/downloaded/.nupkg}`. 

## Creating a new mod
Once the templates have been installed, you can create a new Hollow Knight mod using the command `dotnet new {template}`, where `{template}` may be:
- hkmod: The default, barebones mod template
- hkpreloads: A mod template with some structuring for preloads.
- hksettings: A mod template that implements basic settings.
- hkmpaddon: A template for an add-on for HKMP.

There are several parameters you may pass into the `dotnet` command:
- *--name*: The name of your mod. Should be one word, e.g. "MyMod". Default is "HKMod".
- *--author*: The mod's author, i.e. your name. Default is blank.
- *--description*: A description of the mod. Default is "A Hollow Knight mod."
- *--refsPath*: The location of your project references. This is the folder that contains the `Assembly-CSharp.dll` file. Default is `$(MSBuildProgramFiles32)/Steam/steamapps/common/Hollow Knight/hollow_knight_Data/Managed/`.

### Copying output to multiple locations
If you want to install your build in multiple file locations, you can use this:
- *--refsPath1*: The location of your project references for one installation of Hollow Knight. This is the folder that contains the `Assembly-CSharp.dll` file. Default is `$(MSBuildProgramFiles32)/Steam/steamapps/common/Hollow Knight/hollow_knight_Data/Managed/`.
- *--refsPath2*: The location of your project references for another installation of Hollow Knight. This is the folder that contains the `Assembly-CSharp.dll` file. Default is `$(MSBuildProgramFiles32)/Steam/steamapps/common/Hollow Knight/hollow_knight_Data/Managed/`.


e.g: `dotnet new hkmod --name "MyMod" --author "My Name" --description "Description of my mod." --refsPath "C:\Program Files (x86)\Steam\steamapps\common\Hollow Knight\hollow_knight_Data\Managed\"`

## Uninstall
1.  From a terminal, run `dotnet new -u HKModding.HKMod.Templates`.
