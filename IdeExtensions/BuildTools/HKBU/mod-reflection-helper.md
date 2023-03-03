---
title: Mods References Helper
nav_order: 4
parent: Hollow Knight Build Utils
---

Automatically download MAPI and other mods that depend on it when building.

## How to use

Add the names of the mods you want to depend on to the `<ItemGroup>` in the following way

```xml
<ModReference Include="<Mod Name in ModLinks>" AssemblyName="[Assembly Name]" />
```

Among them, `AssemblyName` is optional, but due to the limitation of MSBuild, if the Mod assembly file name does not match its name on ModLinks, you need to fill in `AssemblyName` by yourself.

For example, Custom Knight, its assembly file name is `CustomKnight.dll` instead of `Custom Knight.dll`, therefore, it needs to be referenced as follows

```xml
<ModReference Include="Custom Knight" AssemblyName="CustomKnight" />
```

**Warning:** The following methods are invalid
```xml
 <ModReference Include="Custom Knight" AssemblyName="*" />
```

## Set game path

Set game paths to use game files directly instead of downloading them again.

HKBuildUtils will look for the game directory in the following order

(1) The contents of hkpath.txt under the project folder or its ancestor folder

(2) `HollowKnightRefs` defined in the project file

**Warning:**
- (1) will be used if both (1) and (2) are found, you can disable (1) and use (2) by using `<DisableOverwriteHollowKnightRefs>true</DisableOverwriteHollowKnightRefs>`
- If neither of them exists or the directory they specify does not exist, it will be considered as a build in CI, and the dependent mods and HKMAPI will be downloaded to the `~ModLibrary` folder under the project folder during the build


