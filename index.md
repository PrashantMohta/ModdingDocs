---
title: Getting Started
nav_order: 1
---
# Hollow Knight Modding Docs

> Please note that this documentation is being written with the latest patch of the game in mind, for older patches you can refer to the documentation at [radiance.host](https://radiance.synthagen.net/apidocs/_images/Getting-Started.html)

## Introduction:  
Hollow Knight (patch 1.5.x.x onwards) is a game built with Unity 2020.2.2 and C# (.NET Framework 4.7.2), this means that a mod in the context of hollow knight is a `dll` or a Dynamic-Link Library that allows players to modify the behavior of the game, fix issues or add new features. This is achieved with the help of a Modding Api that handles the loading of mods and giving us ways of injecting our own code in the middle of normal logic.

The [Hollow Knight Modding Api](https://github.com/hk-modding/api) is based on [MonoMod](https://github.com/MonoMod/MonoMod), a General purpose .NET assembly modding "basework".

If you're just getting started with modding hollow knight, you should start with the [Getting Started Guide](getting-started.md) to get your system setup for modding.  

## Your first mod

  Once you have your system set-up, you can move on to creating [your first mod](your-first-mod.md).  
  
## Deeper dive into modding

The next few sections will give you a better idea of what modding hollow knight looks like beyond simple mods, Covering the concepts, tools & techniques that exist to make the job easier. It goes without saying that the [Unity Scripting Reference](https://docs.unity3d.com/2020.2/Documentation/ScriptReference/) and [Microsoft .NET API browser](https://docs.microsoft.com/en-us/dotnet/api/?view=netframework-4.7.2) are absolutely invaluable references when working within unity. In the context of hollow knight modding though, there are a few more concepts and resources that you want to be looking at. 

 - Concepts
	 - [The Mod Class](mod-baseclass.md)
	 - [The Mod Lifecycle](mod-lifecycle.md) 
	 - [Preloading game objects](preloads.md)
	 - [Logging](logging.md)
	 - [Hooks](Hooks/hooks.md)
	 - [Saving Mod Data](saving-mod-data.md)
	 - [Mod Menu](modmenu.md)
	 - [Hollow Knight Classes](hkclasses.md)
	 - [PlaymakerFSMs](understanding-fsms.md)
	 - [Dependency Mods](dependencymods.md)
	 - [Scripting](#todo-section)

 - Investigative tools
	   <br>These tools can help investigate how a system works in the game so that you can figure out the best way to modify it.
	 - [ILspy](Tools/decompilers.md)
	 - [FSM Viewer](Tools/fsmviewer.md)
	 - [Unity Explorer](https://github.com/sinai-dev/UnityExplorer/blob/master/README.md#features)
	 - [Scene Dump](Tools/scenedump.md)

 - Advanced Concepts
	 - [Reflection](reflection.md)
	 - [IL Hooks](Hooks/ilhooks.md)
	 - [Custom NPC](#todo-section)
	 - [Custom Enemy](#todo-section)
	 - [Custom Scene](#todo-section)

## Publishing and distributing your mod

Once you have created a mod that you want to share with the world, you should look into [publishing your mod on github](#todo-section). This will allow others to look at the source code and get inspired from your creation. They might even contribute to it, making your mod all the better! Once that is done you can share the link of your published github release with all your friends. If you want to add your mod to the mod installer (scarab) then you can [add your mod to the modlinks](#adding-your-mod-to-modlinks).  

## Adding your mod to modlinks

To add your mod to the mod installer:

- [Visit the modlinks page](https://github.com/hk-modding/modlinks/blob/main/ModLinks.xml)
- Instead of forking the entire repository, click the pencil at the top right corner to edit the `Modlinks.xml` file.
- Make your changes to the file.
Example mod manifest:

```xml
<Manifest>
    <!-- Not Optional -->
    <Name>Test Name</Name>
    <Description>Test description</Description>
    <Version>0.0.0.0</Version>
    <Link SHA256="0000000000000000000000000000000000000000000000000000000000000000"><![CDATA[https://download.link]]></Link>
    <Dependencies>
        <Dependency>Another mod's name that this mod depends on</Dependency>
    </Dependencies>
    <Repository>
        <![CDATA[https://github.com/user/repo]]>
    </Repository>
    <!-- Optional -->
    <Issues>
        <![CDATA[https://website/for/bug_reports]]>
    </Issues>
    <Integrations>
        <Integration>Another mod's name, like the dependency list, but not a hard dependency, but extra stuff when it is installed alongside</Integration>
    </Integrations>
    <Tags>
        <Tag>Either of: 'Boss', 'Cosmetic', 'Expansion', 'Gameplay', 'Library', 'Utility'</Tag>
    </Tags>
    <Authors>
        <Author>An author's name. Has no specific format.</Author>
    </Authors>
</Manifest>
```

Example:

```xml
<Manifest>
    <Name>Hallownest-Vocalized</Name>
    <Description>Adds English voice acting to all Hollow Knight's in-game dialogue.</Description>
    <Version>1.0.1.2</Version>
    <Link SHA256="F9529A1D664C1020DD4AFE6A7E06BF77EEA614F5E5079E726CF5F7704CD01EFC">
    <![CDATA[https://github.com/Hallownest-Vocalized/Hallownest-Vocalized/releases/download/0.0.1.2/HKVocal.zip]]>
    </Link>
    <Dependencies>
      <Dependency>Satchel</Dependency>
      <Dependency>Core.FsmUtil</Dependency>
      <Dependency>HKMirror</Dependency>
      <Dependency>Osmi</Dependency>
      <Dependency>SFCore</Dependency>
      <Dependency>FrogCore</Dependency>
      <Dependency>Hallownest-Vocalized-AudioLoader</Dependency>
    </Dependencies>
    <Repository>
      <![CDATA[https://github.com/Hallownest-Vocalized/Hallownest-Vocalized]]>
    </Repository>
    <Tags>
      <Tag>Gameplay</Tag>
    </Tags>
    <Authors>
      <Author>RedFrog</Author>
      <Author>Mulhima</Author>
      <Author>Link459</Author>
      <Author>Clove</Author>
      <Author>Zickles</Author>
    </Authors>
</Manifest>
```

- After editing ensure that
  - The mod name is unique.
  - The version is in the format `0.0.0.0` and without any letters. (not like `v0.0.0.0`)
  - The SHA256 is of the file that is going downloaded by the installer. You can get it from a website like [this](https://emn178.github.io/online-tools/sha256_checksum.html).
- After you are done editing the file
  - Click `Commit Changes...` at the top right of the page.
  - Write a meaningful commit message. For example: `Added {Mod Name}`.
  - Then click propose changes.
  - After clicking "Propose changes," you'll be taken to a new page where you can review your proposed changes.
  - At the top of the page, there should be a button to "Create pull request." Click on it.
  - Once the pull request is created, it will be reviewed by project maintainers or contributors.
  - If the changes are approved, a maintainer will merge the pull request into the modlinks repository.
  - After the pull request is merged, your mod should hopefully appear on the installer!

## Todo-section

This documentation is a work in progress, as such there are many sections that are incomplete currently or are marked with *TODO*, if you're a modder who is interested in contributing to the documentation or even if you simply want to correct a typo, head over to the [github repo](https://github.com/PrashantMohta/ModdingDocs) and raise a pull request.
