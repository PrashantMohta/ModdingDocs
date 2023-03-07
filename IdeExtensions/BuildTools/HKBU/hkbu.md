---
title: Hollow Knight Build Utils
nav_order: 2
parent: IDE Extensions
has_children: true
---

# Hollow Knight Build Utils

[Hollow Knight Build Utils](https://www.nuget.org/packages/HKBuildUtils) is a NuGet package that aims to aid in mod development.

## Features

- [Automatically Pack Mods](modpack.md): Automatically pack the mod into a zip file and calculate the SHA256 value after the build.

- [Mods References Helper](mod-reflection-helper.md): Automatically download HKMAPI and other mods that depend on it when building.

- [Reflect Helper](reflect-helper.md): Allows developers to directly access private fields and private types.

- [Mono Mod Hooks Helper](monomod-hooks-helper.md): Allow developers to use MonoModHooks outside of `MMHOOK_Assembly-CSharp` and `MMHOOK_PlayMaker.dll`.

- Mod Resources Helper.

## Examples
- [Example](https://github.com/HKLab/HKBuildUtils/tree/master/Example/Example/Example)
- [Test Mod](https://github.com/HKLab/HKBuildUtils/tree/master/Example/TestMod/TestMod)
    - Strictly speaking, this is not an example, but it uses almost all functions

## How to use

Add a Nuget package reference to [HKBuildUtils](https://www.nuget.org/packages/HKBuildUtils) like this:
```
dotnet add package HKBuildUtils
```
or
```
<PackageReference Include="HKBuildUtils" Version="*">
  <PrivateAssets>all</PrivateAssets>
  <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
</PackageReference>
```

## Migrating from the mod template made by BadMagic
To do this just a few steps are required:

1. Add a Nuget package reference to [HKBuildUtils](https://www.nuget.org/packages/HKBuildUtils).
2. Remove all `<Reference />` items (for example, remove this: `<Reference Include="UnityEngine.VehiclesModule" />`).
3. Remove `CopyMod` target.
