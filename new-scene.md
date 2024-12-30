# Creating custom Scenes with Unity

## Starting notes

This guide will have you create a Mod that adds a custom Scene that will sit in between the transition from Dirtmouth to the Forgotten Crossroads.

It will be assumed that you know how a Mod can be created, built and then played, as well as how to handle embedded resources.

This guide will follow the mod *[Stories of a HK player - Chapter 2](https://github.com/SFGrenade/StoriesOfaHkPlayer-Ch2)*, as it is one of the smaller mods that use these steps.  
But differences will be made in order to get a result that can be easily tested.

You can check out its source code (and all 3 different branches) to get more detail.

## Requirements

For creating a Mod with a custom Scene the following is necessary:
* A modded Hollow Knight installation
* C# IDE
  * Visual Studio, JetBrains Rider
  * Any text editor alongside the dotnet cli works as well.
* [Unity version 2020.2.2](https://unity.com/releases/editor/whats-new/2020.2.2)
  * While older unity versions also work, using the one of the latest Hollow Knight version is highly recommended.

And for Mod-specific requirements:
* SFCore
  * Needed because of the MonoBehaviours we will use in the custom Scene.
* SFCoreUnity.dll
  * This can be taken from the [SFCore github releases](https://github.com/SFGrenade/SFCore/releases/latest)

## Mod project setup

For the Mod, we will need 2 C# and 1 Unity project.

| Project                    | Purpose                                                                                                         |
|----------------------------|-----------------------------------------------------------------------------------------------------------------|
| MonoBehaviour Project (C#) | This will be the dll that will be put in the Unity's project files to add custom MonoBehaviours on GameObjects. |
| Unity Project              | This will create Asset Bundles that will be loaded by the Final Mod Project.                                    |
| Final Mod Project (C#)     | This will be the actual Mod that will be put in the Game's Mod folder to make the Mod playable.                 |

### Final Mod Project

We start with this one as it's the first step of the mod actually loading in the first place. The MonoBehaviour Project also is derived from this, but more on that later.

First thing to note is that we will need `SFCore` as a dependency mod, mainly for the use of its MonoBehaviours, but the other parts of it will also prove useful.  
So the projects .csproj file should include something like:
```xml
<ItemGroup>
  <Reference Include="$(HollowKnightRefs)/*.dll" />
  <Reference Include="$(HollowKnightRefs)/Mods/SFCore/*.dll" />
</ItemGroup>
```
> This was tested on Windows with JetBrains Rider, if it doesn't work for you, you will probably need to individually list out every dependency dll.

Then in the Mod class we need the following:
```cs
public class MyFirstCustomSceneMod : SaveSettingsMod<SettingsClass>
{
  private AssetBundle _abScenes = null;

  private void LoadAssetBundles()
  {
    Assembly asm = Assembly.GetExecutingAssembly();
    if (_abScenes == null)
    {
      using Stream s = asm.GetManifestResourceStream("MyFirstCustomSceneMod.Resources.my_first_assetbundle");
      if (s != null)
      {
        _abScenes = AssetBundle.LoadFromStream(s);
      }
    }
  }

  public MyFirstCustomSceneMod() : base("My First Custom Scene Mod")
  {
    LoadAssetbundles();

    InitCallbacks();
  }

  public override List<ValueTuple<string, string>> GetPreloadNames()
  {
    return new List<ValueTuple<string, string>>
    {
      // we will populate this later
    };
  }

  public override void Initialize(Dictionary<string, Dictionary<string, GameObject>> preloadedObjects)
  {
    // this will hold on to any preloads for other components to use
    PrefabHolder.Preloaded(preloadedObjects);
  }

  private void InitCallbacks()
  {
    ModHooks.GetPlayerBoolHook += OnGetPlayerBoolHook;
    ModHooks.SetPlayerBoolHook += OnSetPlayerBoolHook;
    ModHooks.LanguageGetHook += OnLanguageGetHook;
    UnityEngine.SceneManagement.SceneManager.activeSceneChanged += OnSceneChanged;
  }
}
```

For saving information, we will use the following class for local save settings:
```cs
public class SettingsClass
{
  // we name the members this way to make it easier for the Get-/SetPlayerBoolHook
  public bool MyFirstCustomSceneMod_VisitedArea = false;
}
```

And to actually use this value, the hooks looks like this:
```cs
private bool OnGetPlayerBoolHook(string target, bool orig)
{
  var tmpField = ReflectionHelper.GetFieldInfo(typeof(SettingsClass), target);
  if (tmpField != null)
  {
    return (bool)tmpField.GetValue(SaveSettings);
  }

  return orig;
}

private bool OnSetPlayerBoolHook(string target, bool orig)
{
  var tmpField = ReflectionHelper.GetFieldInfo(typeof(SettingsClass), target);
  if (tmpField != null)
  {
    tmpField.SetValue(SaveSettings, orig);
  }

  return orig;
}

private string OnLanguageGetHook(string key, string sheet, string orig)
{
  // this hook can be trivialized using `SFCore.Utils.LanguageStrings`
  if (sheet == "Titles" && key == "MyFirstCustomSceneMod_AreaTitle_SUPER")
  {
    return "The";
  }
  if (sheet == "Titles" && key == "MyFirstCustomSceneMod_AreaTitle_MAIN")
  {
    return "Lands";
  }
  if (sheet == "Titles" && key == "MyFirstCustomSceneMod_AreaTitle_SUB")
  {
    return "Between";
  }

  return orig;
}

private void OnSceneChanged(UnityEngine.SceneManagement.Scene from, UnityEngine.SceneManagement.Scene to)
{
  if (to.name == "Town")
  {
    // we arrived in Dirtmouth
    // todo: add code that redirects the well transition to our custom scene
  }
  else if (to.name == "Crossroads_01")
  {
    // we arrived in the Forgotten Crossroads
    // todo: add code that redirects the well transition to our custom scene
  }
}
```

Now to that `PrefabHolder` class and the empty `GetPreloadNames()` method.

For Preloading, it is strongly advised to get as many as your preloads from the same scene, as that will lower the overall time it takes to preload everything.  
Alternatively you can test around and find scenes that are small and easy to preload.

For our custom Scene, we will need:
- an `Area Title Controller` to display a custom area name in our custom Scene
- an `_Managers/PlayMaker Unity 2D` to have `PlayMakerFSM`s work correctly with any NPC name display.

So with that the `GetPreloadNames()` will look like this:
```cs
public override List<ValueTuple<string, string>> GetPreloadNames()
{
  return new List<ValueTuple<string, string>>
  {
    new ValueTuple<string, string>("White_Palace_18", "Area Title Controller"),
    new ValueTuple<string, string>("White_Palace_18", "_Managers/PlayMaker Unity 2D")
  };
}
```

And the `PrefabHolder` class looks like:
```cs
using UObject = UnityEngine.Object;
class PrefabHolder
{
  public static GameObject PopAreaTitleCtrlPrefab { get; private set; }
  public static GameObject PopPmU2dPrefab { get; private set; }

  public static void Preloaded(Dictionary<string, Dictionary<string, GameObject>> preloadedObjects)
  {
    PopAreaTitleCtrlPrefab = UObject.Instantiate(preloadedObjects["White_Palace_18"]["Area Title Controller"]);
    SetInactive(PopAreaTitleCtrlPrefab);
    PopPmU2dPrefab = UObject.Instantiate(preloadedObjects["White_Palace_18"]["_Managers/PlayMaker Unity 2D"]);
    SetInactive(PopPmU2dPrefab);
  }
  private static void SetInactive(GameObject go)
  {
    if (go != null)
    {
      UObject.DontDestroyOnLoad(go);
      go.SetActive(false);
    }
  }
  private static void SetInactive(UObject go)
  {
    if (go != null)
    {
      UObject.DontDestroyOnLoad(go);
    }
  }
}
```

Now to use these 2 preloaded objects.

For the `Area Title Controller` we will create the following `MonoBehaviour`:
```cs
class PatchAreaTitleController : MonoBehaviour
{
  [Range(0, 10)]
  public float Pause = 3f;
  public bool AlwaysVisited = false;
  public bool DisplayRight = false;
  public bool OnlyOnRevisit = false;
  public bool SubArea = true;
  public bool WaitForTrigger = false;
  public string AreaEvent = "";
  public string VisitedBool = "";

  public void Awake()
  {
    GameObject atc = Instantiate(PrefabHolder.PopAreaTitleCtrlPrefab);
    atc.SetActive(false);
    atc.transform.localPosition = transform.position;
    atc.transform.localEulerAngles = transform.eulerAngles;
    atc.transform.localScale = transform.lossyScale;

    PlayMakerFSM atcFsm = atc.LocateMyFSM("Area Title Controller");
    atcFsm.GetFloatVariable("Unvisited Pause").Value = Pause;
    atcFsm.GetFloatVariable("Visited Pause").Value = Pause;

    atcFsm.GetBoolVariable("Always Visited").Value = AlwaysVisited;
    atcFsm.GetBoolVariable("Display Right").Value = DisplayRight;
    atcFsm.GetBoolVariable("Only On Revisit").Value = OnlyOnRevisit;
    atcFsm.GetBoolVariable("Sub Area").Value = SubArea;
    atcFsm.GetBoolVariable("Visited Area").Value = PlayerData.instance.GetBool(VisitedBool);
    atcFsm.GetBoolVariable("Wait for Trigger").Value = WaitForTrigger;

    atcFsm.GetStringVariable("Area Event").Value = AreaEvent;
    atcFsm.GetStringVariable("Visited Bool").Value = VisitedBool;

    atcFsm.GetGameObjectVariable("Area Title").Value = GameObject.Find("Area Title");

    atc.AddComponent<NonBouncer>();
    atc.SetActive(true);

    Destroy(gameObject);
  }
}
```

And for the `_Managers/PlayMaker Unity 2D`, the following `MonoBehaviour` is created:
```cs
class PatchPlayMakerManager : MonoBehaviour
{
  public Transform ManagerTransform;

  public void Awake()
  {
    GameObject tmpPmu2D = Instantiate(PrefabHolder.PopPmU2dPrefab, ManagerTransform);
    tmpPmu2D.SetActive(true);
    tmpPmu2D.name = "PlayMaker Unity 2D";
  }
}
```

With this the Final Mod Project is done for now!

### MonoBehaviour Project

This project will contain the `MonoBehaviour`s of our Final Mod Project, but without the function bodies.  
So like this:
```cs
class PatchAreaTitleController : MonoBehaviour
{
  [Range(0, 10)]
  public float Pause = 3f;
  public bool AlwaysVisited = false;
  public bool DisplayRight = false;
  public bool OnlyOnRevisit = false;
  public bool SubArea = true;
  public bool WaitForTrigger = false;
  public string AreaEvent = "";
  public string VisitedBool = "";

  public void Awake()
  {
  }
}
```
```cs
class PatchPlayMakerManager : MonoBehaviour
{
  public Transform ManagerTransform;

  public void Awake()
  {
  }
}
```

> Do note that both the Final Mod Project and the MonoBehaviour Project have to create Assemblies with the same name and namespaces inside.

And with that the MonoBehaviour Project is ready for Unity.

### Unity Project

The Unity Project should be created as a 2D Project, as that enables us to use 2D components, which will be necessary.

In the Unity Project, we will need:
- The `SFCoreUnity.dll` renamed to `SFCore.dll`
- The `MyFirstCustomSceneMod.dll` from the MonoBehaviour Project
- The following `TutorialScene.obj` file:
  ```obj
  o TutorialScene
  v -30.000000 0.000000 0.000000
  v 0.000000 0.000000 0.000000
  v -30.000000 17.000000 0.000000
  v 0.000000 2.000000 0.000000
  v -30.000000 2.000000 0.000000
  v -30.000000 6.000000 0.000000
  v 0.000000 17.000000 0.000000
  v 0.000000 6.000000 0.000000
  v -2.000000 6.000000 0.000000
  v -2.000000 15.000000 0.000000
  v -28.000000 15.000000 0.000000
  v -28.000000 6.000000 0.000000
  vn -0.0000 -0.0000 -1.0000
  vt 0.000000 0.000000
  s 0
  f 2/1/1 1/1/1 5/1/1 4/1/1
  f 3/1/1 7/1/1 8/1/1 9/1/1 10/1/1 11/1/1 12/1/1 6/1/1
  ```
  > Yes, this is the entire mesh we will use as Terrain for the custom Scene. And since `.obj` files are just plain text files they can be easily edited using notepad or any other text editor.

The folder-structure can look like the following, for easier management:
- Assets
  - _MonoScripts
    - In here, there will be `.cs` source files that are mostly `MonoBehaviour`s copied from Hollow Knight and then adjusted to only contain the members.
  - Assemblies
    - In here, we will put both the (renamed from `SFCoreUnity.dll`) `SFCore.dll` and `MyFirstCustomSceneMod.dll`.
  - Editor
    - In here, there will be `.cs` source files that will add visualization or other functionality to the Unity Editor, potentially regarding some specific MonoBehaviours.
  - Materials
    - In here, every created Material can be stored, be it Texture Materials or Physics Materials.
  - Meshes
    - In here, we will put the `TutorialScene.obj` file.
  - Scenes
    - In here, we will create using Right Click => Create => Scene and call it "MyFirstCustomSceneMod"

For _MonoScripts, we can create the following files:
- `GlobalEnums/MapZone.cs`
  ```cs
  namespace GlobalEnums
  {
    public enum MapZone
    {
      NONE = 0,
      TEST_AREA = 1,
      KINGS_PASS = 2,
      CLIFFS = 3,
      TOWN = 4,
      CROSSROADS = 5,
      GREEN_PATH = 6,
      ROYAL_GARDENS = 7,
      FOG_CANYON = 8,
      WASTES = 9,
      DEEPNEST = 10,
      HIVE = 11,
      BONE_FOREST = 12,
      PALACE_GROUNDS = 13,
      MINES = 14,
      RESTING_GROUNDS = 15,
      CITY = 16,
      DREAM_WORLD = 17,
      COLOSSEUM = 18,
      ABYSS = 19,
      ROYAL_QUARTER = 20,
      WHITE_PALACE = 21,
      SHAMAN_TEMPLE = 22,
      WATERWAYS = 23,
      QUEENS_STATION = 24,
      OUTSKIRTS = 25,
      KINGS_STATION = 26,
      MAGE_TOWER = 27,
      TRAM_UPPER = 28,
      TRAM_LOWER = 29,
      FINAL_BOSS = 30,
      SOUL_SOCIETY = 31,
      ACID_LAKE = 32,
      NOEYES_TEMPLE = 33,
      MONOMON_ARCHIVE = 34,
      MANTIS_VILLAGE = 35,
      RUINED_TRAMWAY = 36,
      DISTANT_VILLAGE = 37,
      ABYSS_DEEP = 38,
      ISMAS_GROVE = 39,
      WYRMSKIN = 40,
      LURIENS_TOWER = 41,
      LOVE_TOWER = 42,
      GLADE = 43,
      BLUE_LAKE = 44,
      PEAK = 45,
      JONI_GRAVE = 46,
      OVERGROWN_MOUND = 47,
      CRYSTAL_MOUND = 48,
      BEASTS_DEN = 49,
      GODS_GLORY = 50,
      GODSEEKER_WASTE = 51,
    }
  }
  ```
- `GlobalEnums/SceneType.cs`
  ```cs
  namespace GlobalEnums
  {
    public enum SceneType
    {
      GAMEPLAY = 0,
      MENU = 1,
      LOADING = 2,
      CUTSCENE = 3,
      TEST = 4,
    }
  }
  ```
- `ReplacementStuff/AudioMixerSnapshot.cs`
  ```cs
  public class AudioMixerSnapshot {}
  ```
- `ReplacementStuff/PlayMakerFSM.cs`
  ```cs
  public class PlayMakerFSM {}
  ```
- `CameraLockArea.cs`
  ```cs
  using UnityEngine;

  public class CameraLockArea : MonoBehaviour
  {
    public float cameraXMin;
    public float cameraYMin;
    public float cameraXMax;
    public float cameraYMax;
    public bool preventLookUp;
    public bool preventLookDown;
    public bool maxPriority;
  }
  ```
- `HazardRespawnMarker.cs`
  ```cs
  using UnityEngine;

  public class HazardRespawnMarker : MonoBehaviour
  {
    public bool respawnFacingRight;
    public bool drawDebugRays;
  }
  ```
- `HazardRespawnTrigger.cs`
  ```cs
  using UnityEngine;

  public class HazardRespawnTrigger : MonoBehaviour
  {
    public HazardRespawnMarker respawnMarker;
    public bool fireOnce;
  }
  ```
- `NonBouncer.cs`
  ```cs
  using UnityEngine;

  public class NonBouncer : MonoBehaviour
  {
    public bool active;
  }
  ```
- `RealHazardType.cs`
  ```cs
  public enum RealHazardType
  {
    NON_HAZARD = 0,
    NORMAL,
    SPIKES,
    ACID,
    LAVA,
    PIT
  }
  ```
- `RespawnMarker.cs`
  ```cs
  using UnityEngine;

  public class RespawnMarker : MonoBehaviour
  {
    public bool respawnFacingRight;
  }
  ```
- `SceneLoadVisualizations.cs`
  ```cs
  using UnityEngine;

  public class GameManager : MonoBehaviour
  {
    public enum SceneLoadVisualizations
    {
      Default = 0,
      Custom = -1,
      Dream = 1,
      Colosseum = 2,
      GrimmDream = 3,
      ContinueFromSave = 4,
      GodsAndGlory = 5
    }
  }
  ```
- `TransitionPoint.cs`
  ```cs
  using UnityEngine;

  public class TransitionPoint : MonoBehaviour
  {
    public bool isADoor = false;
    [HideInInspector]
    public bool dontWalkOutOfDoor = false;
    [HideInInspector]
    public float entryDelay = 0.0f;
    public bool alwaysEnterRight = false;
    public bool alwaysEnterLeft = false;
    public bool hardLandOnExit = false;
    public string targetScene;
    public string entryPoint;
    [HideInInspector]
    public Vector2 entryOffset = new Vector2(0.0f, 0.0f);
    [HideInInspector]
    public PlayMakerFSM customFadeFSM = null;
    [HideInInspector]
    public bool nonHazardGate = false;
    public HazardRespawnMarker respawnMarker;
    [HideInInspector]
    public AudioMixerSnapshot atmosSnapshot = null;
    [HideInInspector]
    public AudioMixerSnapshot enviroSnapshot = null;
    [HideInInspector]
    public AudioMixerSnapshot actorSnapshot = null;
    [HideInInspector]
    public AudioMixerSnapshot musicSnapshot = null;
    public GameManager.SceneLoadVisualizations sceneLoadVisualization = GameManager.SceneLoadVisualizations.Default;
    [HideInInspector]
    public bool customFade = false;
    [HideInInspector]
    public bool forceWaitFetch = false;
  }
  ```











---
---
---
---
---
OLD STUFF HERE
---
---
---
---
---

4. Add :code:`GetPlayerBoolHook`, :code:`LanguageGetHook` & :code:`UnityEngine.SceneManagement.SceneManager.activeSceneChanged`

5. The :code:`GetPlayerBoolHook` will only listen to a bool that we will call :code:`CustomSceneTutorial_VisitedArea` and will always return false

```c#
private bool OnGetPlayerBoolHook(string target)
{
  if (target == "CustomSceneTutorial_VisitedArea") return false;
  return PlayerData.instance.GetBoolInternal(target);
}
```

6. The :code:`LanguageGetHook` will only listen to 3 strings that start with :code:`"CustomSceneTutorial_AreaTitle_"`

```c#
private string OnLanguageGetHook(string key, string sheet)
{
  if (sheet == "Titles" && key == "CustomSceneTutorial_AreaTitle_SUPER") return "Testing";
  else if (sheet == "Titles" && key == "CustomSceneTutorial_AreaTitle_MAIN") return "area";
  else if (sheet == "Titles" && key == "CustomSceneTutorial_AreaTitle_SUB") return "of doom";
  return Language.Language.GetInternal(key, sheet);
}
```

7. The :code:`UnityEngine.SceneManagement.SceneManager.activeSceneChanged` will do nothing for now

```c#
private void OnSceneChanged(Scene from, Scene to)
{
}
```

8. For the ease of simplicity add the preloads "Area Title Controller" & "_SceneManager" from "White_Palace_18"

```c#
public override List<ValueTuple<string, string>> GetPreloadNames()
{
  return new List<ValueTuple<string, string>>
  {
    new ValueTuple<string, string>("White_Palace_18", "Area Title Controller"),
    new ValueTuple<string, string>("White_Palace_18", "_SceneManager"),
    new ValueTuple<string, string>("White_Palace_18", "_Managers/PlayMaker Unity 2D")
  };
}
```

9. This also means storing these preloaded GOs in your mod class

10. For later use in Unity, also add `a script for inserting an AreaTitleController <https://github.com/SFGrenade/TestOfTeamwork/blob/master/MonoBehaviours/Patcher/PatchAreaTitleController.cs>`_, `a script to insert a SceneManager<https://github.com/SFGrenade/TestOfTeamwork/blob/master/MonoBehaviours/Patcher/PatchSceneManager.cs>`_ and `a script to insert a PlayMaker Manager<https://github.com/SFGrenade/TestOfTeamwork/blob/master/MonoBehaviours/Patcher/PatchPlayMakerManager.cs>`_

11. Also for later use in Unity, add a script for setting the correct width and height of the scene

```c#
class PatchTilemapSize : MonoBehaviour
{
  public int width = 30;
  public int height = 17;

  public void Awake()
  {
    On.GameManager.RefreshTilemapInfo += OnGameManagerRefreshTilemapInfo;
  }

  public void OnDestroy()
  {
    On.GameManager.RefreshTilemapInfo -= OnGameManagerRefreshTilemapInfo;
  }

  private void OnGameManagerRefreshTilemapInfo(On.GameManager.orig_RefreshTilemapInfo orig, GameManager self, string targetScene)
  {
    orig(self, targetScene);
    if (targetScene == gameObject.scene.name)
    {
        self.tilemap.width = width;
        self.tilemap.height = height;
        self.sceneWidth = width;
        self.sceneHeight = height;
        FindObjectOfType<GameMap>().SetManualTilemap(0, 0, width, height);
    }
  }
}
```

## Preparation for Unity
^^^^^^^^^^^^^^^^^^^^^

12. Create a new C# class library project using Unity. Use the one labelled "Class library (.NET Framework)".

13. Give it a name (i suggest the same from before, but with :code:`"Scripts"` behind it) and select .NET Framework 3.5

14. Add ONLY the required unity assemblies as references

15. Copy only the MonoBehaviour classes from before into this new project

16. You can remove all functions from these classes, only the member variables are important

.. note::
  For member variables that are of enum types, you can use other enums that have the same ranges covered as seen in the PatchSceneManager class.

17. Build this MonoBehaviour-Only project and copy the DLL

Unity project setup
^^^^^^^^^^^^^^^^^^^
18. Create a new project using Unity. As a template choose the 3D template. The name is irrelevant.

19. Make a few folders for organizing assets.

.. figure:: resources/newscene/unityfolders.png
  :scale: 35 %

  Figure 1: My personal assortment of folders to organize assets.

20. Paste the copied DLL from before into the :code:`Assemblies` folder, also add the :code:`SFCoreUnity.dll` assembly

.. note::
  Don't forget to rename :code:`SFCoreUnity.dll` to :code:`SFCore.dll`

21. Create a scene in unity

22. Change the lighting of the scene

.. figure:: resources/newscene/unitylighting.png
  :scale: 35 %

  Figure 2: Lighting settings that are good to use

23. Add your terrain meshes, put colliders on them (either :code:`EdgeCollider2D`s or :code:`PolygonCollider2D`s) and put them on layer 8 (aka the terrain layer)

.. note::
  This can utilize custom made meshes in programs like Blender

24. On these mesh GameObjects, add the :code:`SceneMapPatcher` component from SFCore and give it a black texture to use.

25. Behind everything (with a global Z position of around 7), it is good to add a BlurPlane with a :code:`MeshFilter`, :code:`MeshRenderer` and :code:`BluePlanePatcher` component

26. Add decorations, sprites should have the :code:`SpritePatcher` component on them or above them in the hierarchy.

27. Add a GameObject called :code:`__Initializer` with a :code:`PatchAreaTitleController` :code:`PatchSceneManager`, :code:`PatchPlayMakerManager` & :code:`PatchTilemapSize` component

28. The :code:`PatchAreaTitleController` will be set as a sub area with the area event :code:`CustomSceneTutorial_AreaTitle` and the visited bool :code:`CustomSceneTutorial_VisitedArea`

29. The :code:`PatchSceneManager` can be adjusted to ones liking

30. The :code:`PatchPlayMakerManager` should be given a transform of a GameObject called :code:`_Managers`

31. The :code:`PatchTilemapSize` should be given the width & height of the custom scene

32. Add an entry & exit by adding a GameObject with a Collder set as a trigger and a :code:`TransitionPoint` component, which can be added as a simple :code:`.cs` file in the :code:`_MonoBehaviours` folder in the assets

33. Build the assetbundles and include them in the first C# project as embedded resources

34. In the mod, load the assetbundles in either the constructor or the :code:`Initialize` function

Accessing the custom scene
^^^^^^^^^^^^^^^^^^^^^^^^^^

35. in :code:`UnityEngine.SceneManagement.SceneManager.activeSceneChanged` that did nothing until now, add code to create a GameObject with a :code:`TransitionPoint` wherever you want to access your scene from

36. Build the mod

37. Enjoy your first empty custom scene!
