---
title: Reflect Helper
nav_order: 1
parent: Hollow Knight Build Utils
---

Allows developers to directly access private fields and private types

**If you need to use this feature, please add a reference to the Fody package**


## How to use

Use `<ReflectHelper Include="<AssemblyName>" />` to indicate which assemblies need to construct Reflect Helper

> `<AssemblyName>` can use assemblies explicitly referenced in project files using `<Reference>` and MAPI assemblies and other Mods assemblies inserted by HKBuildUtils

For example:

```xml
<ReflectHelper Include="UnityEngine.CoreModule" />
<ReflectHelper Include="Assembly-CSharp" />
```

They indicate the Reflect Helper that generates `UnityEngine.CoreModule` and `Assembly-CSharp`

They can be used like

```cs
HeroControllerR reflect = HeroController.instance.Reflect();
HeroController orig = reflect.ToOriginal();
```
```cs
HeroControllerR reflect = (HeroControllerR)(object)HeroController.instance;
HeroController orig = (HeroController)(object)reflect;
```
```cs
HeroControllerR reflect = HeroControllerR.instance;
```
```cs
ModLoaderR.TryAddModInstance(typeof(TestModMod), new ModInstanceR()
{
   Enabled = true,
   Mod = this,
   Name = "Hello, World!This is Test Mod"
}); 
```

#### Compare with HKMirror

Advantage:

- No need to add additional references
- Can be used with almost any assembly
- Has most of the private types
- Custom operator operations with primitive types
- Primitive object instances can be created directly using `new`. For example, `new ModInstanceR()` will create an instance of `ModLoader.ModInstance`
- Supports generics

Defect:

- No quick access to singleton objects
- Does not carry HookHelper

#### Compare with [HKReflect](https://github.com/Clazex/HKReflect)
- TODO (not released yet)
