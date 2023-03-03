---
title: Mono Mod Hooks Helper
nav_order: 2
parent: Hollow Knight Build Utils
---

Allow developers to use MonoModHooks outside of `MMHOOK_Assembly-CSharp` and `MMHOOK_PlayMaker.dll`

They are automatically generated and automatically referenced at build

**If you need to use this feature, please add a reference to the Fody package**

## How to use

Use `<MonoModHook Include="<AssemblyName>" />` to indicate which assemblies need to construct MonoModHookHelper

> `<AssemblyName>` can use assemblies explicitly referenced in project files using `<Reference>` and MAPI assemblies and other Mods assemblies inserted by HKBuildUtils

Then you can use them like [OnHook](/ModdingDocs/Hooks/onhooks.md) and [ILHook](/ModdingDocs/Hooks/ilhooks.md)

For example:

```xml
<MonoModHook Include="UnityEngine.CoreModule" />
<MonoModHook Include="Assembly-CSharp" />
```

They indicate the MonoHookHelper that generates `UnityEngine.CoreModule` and `Assembly-CSharp`

To avoid errors, this automatically disables the reference to `MMHOOK_Assembly-CSharp` that comes with MAPI

MonoModHookHelper (including those that come with MAPI) will be merged into the mod assembly by Fody in the build, no need to publish additional files
