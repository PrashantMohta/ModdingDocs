---
title: Satchel
nav_order: 1
parent: Dependency Mods
has_children: true
---

# Satchel

A Library mod that aims to make modding more accessible by providing simpler ways of achieving complex functionality, also has some Utilities for general things.

## Features

- [BetterMenus](BetterMenus/better-menus.md) : An easier way to implement ICustomMenuMod.
  - Only asks the user to return a list of all the menu elements they want while the menu builder and such are abstracted away.
  - Provides events and methods that allows you to edit/hide elements at runtime.
  - See [example](https://github.com/PrashantMohta/Satchel/tree/master/BetterMenus/Example)

- Custom UI and game elements
	- CustomArrowPrompt
	- CustomBigItemGet
	- CustomDialoguePrompts
	- CustomDreamNailPrompts
	- CustomEnemy
	- CustomMap
	- CustomSaveSlots
	- CustomScene
	- CustomShiny
	
- Unity Monobehaviours to Aid development within unity
	- AlertRangeMarker
	- ChangeMeshColor
	- CustomEnemyMarker
	- SpriteRendererMaterial
	
- FUtils - FSM related utilities
	- FSMUtility
	- Interceptor
	- Serialiser
	- Extractor
	
- Other General utilities and helpers
	- AnimationUtils
	- AssemblyUtils
	- CoroutineHelper
	- EnemyUtils
	- GameObjectUtils
	- IoUtils
	- PlayerDataUtils
	- SceneUtils
	- SpriteUtils
	- TextureUtils
	- WavUtils
	
- Easy Reflection
	- GameManager
	- HeroController