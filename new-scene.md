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
    // get the transition point in the well
    var tp = to.Find("bot1").GetComponent<TransitionPoint>();
    // we want to get to our custom scene
    tp.targetScene = "MyFirstCustomSceneMod";
    // we'll enter our custom scene from the left
    tp.entryPoint = "left1";
  }
  else if (to.name == "Crossroads_01")
  {
    // we arrived in the Forgotten Crossroads
    // get the transition point in the well
    var tp = to.Find("top1").GetComponent<TransitionPoint>();
    // we want to get to our custom scene
    tp.targetScene = "MyFirstCustomSceneMod";
    // we'll enter our custom scene from the right
    tp.entryPoint = "right1";
  }
}
```
> Note: For Transitions, there are the options of naming them `top#`, `left#`, `right#`, `bot#` and `door#`

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

And for now, temporarily, a 3rd MonoBehaviour is needed, it will be added to SFCore in the future though, so check first if it isn't already available:
```cs
public class PatchTileMap : MonoBehaviour
{
  public int width;
  public int height;
  public int columns;
  public int rows;
  public int partSizeX;
  public int partSizeY;
  public PhysicsMaterial2D physicsMaterial2D;
  public GameObject renderData;

  private void Awake()
  {
    var tileMap = gameObject.AddComponent<tk2dTileMap>();
    tileMap.renderData = renderData;
    tileMap.width = width;
    tileMap.height = height;
    tileMap.partitionSizeX = partSizeX;
    tileMap.partitionSizeY = partSizeY;
    tileMap.Layers = new Layer[]
    {
      new Layer(0, width, height, partSizeX, partSizeY)
      {
        gameObject = renderData.transform.GetChild(0).gameObject,
        numColumns = columns,
        numRows = rows
      }
    };
    tileMap.ColorChannel = new ColorChannel(width, height, partSizeX, partSizeY)
    {
      clearColor = new Color(1, 1, 1, 1),
      numColumns = columns,
      numRows = rows
    };
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
```cs
public class PatchTileMap : MonoBehaviour
{
  public int width;
  public int height;
  public int columns;
  public int rows;
  public int partSizeX;
  public int partSizeY;
  public PhysicsMaterial2D physicsMaterial2D;
  public GameObject renderData;
}
```

> Do note that both the Final Mod Project and the MonoBehaviour Project have to create Assemblies with the same name and namespaces inside.

And with that the MonoBehaviour Project is ready for Unity.

### Unity Project

The Unity Project should be created as a 2D Project, as that enables us to use 2D components, which will be necessary.

Starting, we want to add the Tags and Layers Hollow Knight has to the project, to make it easy to tag and layer GameObjects accurately.

Take this data and copy and paste it into `{Unity Project Folder}/ProjectSettings/TagMamager.asset`:
```
%YAML 1.1
%TAG !u! tag:unity3d.com,2011:
--- !u!78 &1
TagManager:
  serializedVersion: 2
  tags:
  - TileMap
  - GameManager
  - BlackOverlay
  - HeroBox
  - Nail Attack
  - RespawnPoint
  - HeroWalkable
  - SceneManager
  - HeroLight
  - Battle Gate
  - Battle Scene
  - CameraParent
  - Terrain
  - Canvas
  - UIManager
  - Hero Spell
  - Enemy Message
  - Orb Target
  - Vignette
  - RespawnTrigger
  - Boss Corpse
  - Heart Piece
  - TransitionGate
  - UI Soul Orb
  - Shade Marker
  - Hud Camera
  - Cinematic
  - Roar
  - Stag Grate
  - Platform
  - Boss
  - GeoCounter
  - CameraTarget
  - StagMapMarker
  - HeroFootsteps
  - Save Icon
  - HeroLightMain
  - Beta End
  - Shop Window
  - Journal Up Msg
  - Charms Pane
  - Inventory Top
  - Charm Get Msg
  - Acid
  - Soul Vessels
  - Teleplane
  - Water Surface
  - Relic Get Msg
  - Fireball Safe
  - Baby Centipede
  - Knight Hatchling
  - Dream Attack
  - Infected Flag
  - Ghost Warrior NPC
  - Extra Tag
  - Dream Plant
  - Dream Orb
  - Geo
  - Sharp Shadow
  - Boss Attack
  - Nail Beam
  - Grub Bottle
  - Colosseum Manager
  - Wall Breaker
  - Ignore Hatchling
  - Hatchling Magnet
  - Spell Vulnerable
  - Hopper
  - Set Extrapolate
  - Orbit Shield
  - Grimmchild
  - WindyGrass
  - Weaverling
  layers:
  - Default
  - TransparentFX
  - Ignore Raycast
  - 
  - Water
  - UI
  - 
  - 
  - Terrain
  - Player
  - Transition Gates
  - Enemies
  - Projectiles
  - Hero Detector
  - Terrain Detector
  - Enemy Detector
  - Item
  - Hero Attack
  - Particle
  - Interactive Object
  - Hero Box
  - Grass
  - Enemy Attack
  - Damage All
  - Bouncer
  - Soft Terrain
  - Corpse
  - UGUI
  - Hero Only
  - ActiveRegion
  - Map Pin
  - Orbit Shield
  m_SortingLayers:
  - name: Default
    uniqueID: 0
    locked: 0
  - name: Far BG 2
    uniqueID: 3315419377
    locked: 0
  - name: Far BG 1
    uniqueID: 1459018367
    locked: 0
  - name: Mid BG
    uniqueID: 4015848369
    locked: 0
  - name: Immediate BG
    uniqueID: 2917268371
    locked: 0
  - name: Actors
    uniqueID: 1270309357
    locked: 0
  - name: Player
    uniqueID: 3557629463
    locked: 0
  - name: Tiles
    uniqueID: 3868594333
    locked: 0
  - name: MID Dressing
    uniqueID: 3784110789
    locked: 0
  - name: Immediate FG
    uniqueID: 31172181
    locked: 0
  - name: Far FG
    uniqueID: 2577183099
    locked: 0
  - name: Vignette
    uniqueID: 1038907033
    locked: 0
  - name: Over
    uniqueID: 3945752401
    locked: 0
  - name: HUD
    uniqueID: 629535577
    locked: 0
```

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
    - We can already create a `Physics Material 2D` with `Friction`: `0.2` and `Bouncines`: `0`
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

For Editor, we can create the following files:
- `CameraLockAreaEditor.cs`  
  This script displays which area the in-game camera will show when you enter a `CameraLockArea`
  ```cs
  using UnityEngine;
  using System.Collections;
  using UnityEditor;

  [CustomEditor(typeof(CameraLockArea))]
  public class CameraLockAreaEditor : Editor 
  {
    void OnSceneGUI()
    {
      var cla = target as CameraLockArea;
      var transform = cla.transform;
      var positions = new Vector3[5];
      positions[0] = new Vector3(cla.cameraXMin - 14.6f, cla.cameraYMin - 8.3f);
      positions[1] = new Vector3(cla.cameraXMax + 14.6f, cla.cameraYMin - 8.3f);
      positions[2] = new Vector3(cla.cameraXMax + 14.6f, cla.cameraYMax + 8.3f);
      positions[3] = new Vector3(cla.cameraXMin - 14.6f, cla.cameraYMax + 8.3f);
      positions[4] = new Vector3(cla.cameraXMin - 14.6f, cla.cameraYMin - 8.3f);
      Handles.DrawPolyLine(positions);
    }
  }
  ```
- `CameraModeSwitch.cs`  
  This script allows you to easily switch all `Camera`s in a Scene to either Orthographic or Perspective transparency sort mode, which is usefull since Hollow Knight's camera is a 3d camera but in Orthographic transparency sort mode.
  ```cs
  using UnityEngine;
  using UnityEditor;

  public static class CameraModeSwitch
  {
    [MenuItem("Camera/Orthographic")]
    static public void OrthographicCamera()
    {
      foreach (var cam in GameObject.FindObjectsOfType<Camera>())
        cam.transparencySortMode = TransparencySortMode.Orthographic;
    }
    [MenuItem("Camera/Perspective")]
    static public void PerspectiveCamera()
    {
      foreach (var cam in GameObject.FindObjectsOfType<Camera>())
        cam.transparencySortMode = TransparencySortMode.Default;
    }
  }
  ```
- `CreateAssetBundles.cs`  
  This script allows you to easily build Asset Bundles for windows, which will work for all platforms right up until you use custom shaders.
  ```cs
  using UnityEditor;
  using System.IO;

  public class CreateAssetBundles
  {
    [MenuItem("Build AssetBundles/Build AssetBundles Compressed")]
    static void BuildAllAssetBundlesCompressed()
    {
      string assetBundleDirectory = "Assets/AssetBundles";
      if(!Directory.Exists(assetBundleDirectory))
      {
        Directory.CreateDirectory(assetBundleDirectory);
      }
      BuildPipeline.BuildAssetBundles(assetBundleDirectory, 
                                      BuildAssetBundleOptions.None, 
                                      BuildTarget.StandaloneWindows);
    }
    
    [MenuItem("Build AssetBundles/Build AssetBundles Uncompressed")]
    static void BuildAllAssetBundlesUncompressed()
    {
      string assetBundleDirectory = "Assets/AssetBundles";
      if(!Directory.Exists(assetBundleDirectory))
      {
        Directory.CreateDirectory(assetBundleDirectory);
      }
      BuildPipeline.BuildAssetBundles(assetBundleDirectory, 
                                      BuildAssetBundleOptions.UncompressedAssetBundle, 
                                      BuildTarget.StandaloneWindows);
    }
  }
  ```
- `MeshCollisionCreator.cs`  
  This script allows you to easily create collisions for meshes.
  ```cs
  using UnityEngine;
  using System.Collections;
  using UnityEditor;
  using System.Collections.Generic;
  using System.Linq;
  using System;

  public class MeshCollisionCreator : Editor 
  {
    [MenuItem("CONTEXT/MeshFilter/Create Collision")]
    public static void GenerateCollision(MenuCommand menuCommand)
    {
      Debug.Log("GenerateMap called");

      if ((menuCommand.context as MeshFilter).gameObject.GetComponent<PolygonCollider2D>() == null) {
        (menuCommand.context as MeshFilter).gameObject.AddComponent<PolygonCollider2D>();
      }
      
      CreatePolygon2DColliderPoints((menuCommand.context as MeshFilter), (menuCommand.context as MeshFilter).gameObject.GetComponent<PolygonCollider2D>());

      Debug.Log("GenerateMap ended");
    }
    
    #region Helper
    private static void CreatePolygon2DColliderPoints(MeshFilter filter, PolygonCollider2D polyCollider)
    {
      var edges = BuildEdgesFromMesh(filter);
      var paths = BuildColliderPaths(edges);
      ApplyPathsToPolygonCollider(polyCollider, paths);
    }

    private static Dictionary<Edge2D, int> BuildEdgesFromMesh(MeshFilter filter)
    {
      var mesh = filter.sharedMesh;

      if (mesh == null)
        return null;

      var verts = mesh.vertices;
      var tris = mesh.triangles;
      var edges = new Dictionary<Edge2D, int>();

      for (int i = 0; i < tris.Length - 2; i += 3) {

        var faceVert1 = verts[tris[i]];
        var faceVert2 = verts[tris[i + 1]];
        var faceVert3 = verts[tris[i + 2]];

        Edge2D[] faceEdges;
        faceEdges = new Edge2D[] {
          new Edge2D{ a = faceVert1, b = faceVert2 },
          new Edge2D{ a = faceVert2, b = faceVert3 },
          new Edge2D{ a = faceVert3, b = faceVert1 },
        };

        foreach(var edge in faceEdges) {
          if (edges.ContainsKey(edge))
            edges[edge]++;
          else
            edges[edge] = 1;
        }
      }

      return edges;
    }

    private static List<Vector2[]> BuildColliderPaths(Dictionary<Edge2D, int> allEdges)
    {
      if (allEdges == null)
        return null;

      var outerEdges = GetOuterEdges(allEdges);

      var paths = new List<List<Edge2D>>();
      List<Edge2D> path = null;

      while (outerEdges.Count > 0) {
        if (path == null) {
          path = new List<Edge2D>();
          path.Add (outerEdges[0]);
          paths.Add (path);

          outerEdges.RemoveAt(0);
        }
        bool foundAtLeastOneEdge = false;
        int i = 0;
        while (i < outerEdges.Count) {
          var edge = outerEdges [i];
          bool removeEdgeFromOuter = false;

          if (edge.b == path[0].a) {
            path.Insert (0, edge);
            removeEdgeFromOuter = true;
          }
          else if (edge.a == path[path.Count - 1].b) {
            path.Add(edge);
            removeEdgeFromOuter = true;
          }

          if (removeEdgeFromOuter) {
            foundAtLeastOneEdge = true;
            outerEdges.RemoveAt(i);
          } else
            i++;
        }
        //If we didn't find at least one edge, then the remaining outer edges must belong to a different path
        if (!foundAtLeastOneEdge)
          path = null;
      }
      var cleanedPaths = new List<Vector2[]>();
      foreach(var builtPath in paths) {
        var coords = new List<Vector2>();

        foreach(var edge in builtPath)
          coords.Add (edge.a);

        cleanedPaths.Add (CoordinatesCleaned(coords));
      }
      return cleanedPaths;
    }

    private static void ApplyPathsToPolygonCollider(PolygonCollider2D polyCollider, List<Vector2[]> paths)
    {
      if (paths == null)
        return;

      polyCollider.pathCount = paths.Count;
      for (int i = 0; i < paths.Count; i++) {
        var path = paths [i];
        polyCollider.SetPath(i, path);
      }
    }

    private static List<Edge2D> GetOuterEdges(Dictionary<Edge2D, int> allEdges)
    {
      var outerEdges = new List<Edge2D>();

      foreach(var edge in allEdges.Keys) {
        var numSharedFaces = allEdges[edge];
        if (numSharedFaces == 1)
          outerEdges.Add (edge);
      }

      return outerEdges;
    }

    private static bool CoordinatesFormLine(Vector2 a, Vector2 b, Vector2 c)
    {
      //If the area of a triangle created from three points is zero, they must be in a line.
      float area = a.x * ( b.y - c.y ) + b.x * ( c.y - a.y ) + c.x * ( a.y - b.y );
      return Mathf.Approximately(area, 0f);
    }

    private static Vector2[] CoordinatesCleaned(List<Vector2> coordinates)
    {
      List<Vector2> coordinatesCleaned = new List<Vector2> ();
      coordinatesCleaned.Add (coordinates [0]);

      var lastAddedIndex = 0;

      for (int i = 1; i < coordinates.Count; i++) {
        var coordinate = coordinates [i];

        Vector2 lastAddedCoordinate = coordinates [lastAddedIndex];
        Vector2 nextCoordinate = (i + 1 >= coordinates.Count) ? coordinates[0] : coordinates [i + 1];

        if (!CoordinatesFormLine(lastAddedCoordinate, coordinate, nextCoordinate)) {
          coordinatesCleaned.Add (coordinate);
          lastAddedIndex = i;
        }
      }
      return coordinatesCleaned.ToArray ();
    }

    #region Nested
    struct Edge2D {
      public Vector2 a;
      public Vector2 b;

      public override bool Equals (object obj)
      {
        if (obj is Edge2D) {
          var edge = (Edge2D)obj;
          //An edge is equal regardless of which order it's points are in
          return (edge.a == a && edge.b == b) || (edge.b == a && edge.a == b);
        }
        return false;

      }

      public override int GetHashCode ()
      {
        return a.GetHashCode() ^ b.GetHashCode();
      }

      public override string ToString ()
      {
        return string.Format ("["+a.x+","+a.y+"->"+b.x+","+b.y+"]");
      }
    }
    #endregion
    #endregion
  }
  ```
- `SceneManagerPatcherEditor.cs`  
  This script allows you to easily work with the `SceneManagerPatcher` `MonoBehaviour` from SFCore.
  ```cs
  using UnityEngine;
  using System.Collections;
  using UnityEditor;
  using SFCore.MonoBehaviours;

  [CustomEditor(typeof(SceneManagerPatcher))]
  [CanEditMultipleObjects]
  public class SceneManagerPatcherEditor : Editor
  {
    string[] _musicChoices = new []
    {
      "Normal",
      "Normal Alt",
      "Normal Soft",
      "Normal Softer",
      "Normal Flange",
      "Normal Flangier",
      "Action",
      "Action and Sub",
      "Sub Area",
      "Silent",
      "Silent Flange",
      "Off",
      "Tension Only",
      "Normal - Gramaphone",
      "Action Only",
      "Main Only",
      "HK Decline 2",
      "HK Decline 3",
      "HK Decline 4",
      "HK Decline 5",
      "HK Decline 6"
    };
    string[] _atmosChoices = new []
    {
      "at None",
      "at Cave",
      "at Surface",
      "at Surface Interior",
      "at Surface Basement",
      "at Surface Nook",
      "at Rainy Indoors",
      "at Rainy Outdoors",
      "at Distant Rain",
      "at Distant Rain Room",
      "at Greenpath",
      "at Queens Gardens",
      "at Fungus",
      "at Fog Canyon",
      "at Waterways Flowing",
      "at Waterways",
      "at Greenpath Interior",
      "at Fog Canyon Minor",
      "at Mines Crystal",
      "at Mines Machinery",
      "at Deepnest",
      "at Deepnest Quiet",
      "at Wind Tunnel",
      "at Misc Wind"
    };
    string[] _enviroChoices = new []
    {
      "en Cave",
      "en Spa",
      "en Cliffs",
      "en Room",
      "en Arena",
      "en Sewerpipe",
      "en Fog Canyon",
      "en Dream",
      "en Silent"
    };
    string[] _actorChoices = new []
    {
      "On",
      "Off"
    };
    string[] _shadeChoices = new []
    {
      "Away",
      "Close"
    };

    public override void OnInspectorGUI()
    {
      serializedObject.Update();

      EditorGUILayout.PropertyField(serializedObject.FindProperty("mapZone"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("isWindy"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("isTremorZone"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("environmentType"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("darknessLevel"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("noLantern"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("saturation"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("ignorePlatformSaturationModifiers"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("redChannel"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("greenChannel"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("blueChannel"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("defaultColor"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("defaultIntensity"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("heroLightColor"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("noParticles"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("overrideParticlesWith"), true);

      // --- AUDIO ---------------------------------------

      EditorGUILayout.PropertyField(serializedObject.FindProperty("AtmosCueSet"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("AtmosCueSnapshotIndex"), new GUIContent("Atmos Cue Snapshot"));
      serializedObject.FindProperty("AtmosCueSnapshotName").stringValue = _atmosChoices[serializedObject.FindProperty("AtmosCueSnapshotIndex").intValue];
      EditorGUILayout.PropertyField(serializedObject.FindProperty("AtmosCueIsChannelEnabled"), true);

      EditorGUILayout.PropertyField(serializedObject.FindProperty("MusicCueSet"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("MusicCueSnapshotIndex"), new GUIContent("Music Cue Snapshot"));
      serializedObject.FindProperty("MusicCueSnapshotName").stringValue = _musicChoices[serializedObject.FindProperty("MusicCueSnapshotIndex").intValue];
      EditorGUILayout.PropertyField(serializedObject.FindProperty("MusicCueChannelInfoClips"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("MusicCueChannelInfoSyncs"), true);

      EditorGUILayout.PropertyField(serializedObject.FindProperty("InfectedMusicCueSet"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("InfectedMusicCueSnapshotIndex"), new GUIContent("Infected Music Cue Snapshot"));
      serializedObject.FindProperty("InfectedMusicCueSnapshotName").stringValue = _musicChoices[serializedObject.FindProperty("InfectedMusicCueSnapshotIndex").intValue];
      EditorGUILayout.PropertyField(serializedObject.FindProperty("InfectedMusicCueChannelInfoClips"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("InfectedMusicCueChannelInfoSyncs"), true);

      // --- AUDIO ---------------------------------------

      EditorGUILayout.PropertyField(serializedObject.FindProperty("musicDelayTime"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("musicTransitionTime"), true);

      // --- AUDIO ---------------------------------------
      
      EditorGUILayout.PropertyField(serializedObject.FindProperty("MsSnapshotIndex"), new GUIContent("Music Snapshot"));
      serializedObject.FindProperty("MsSnapshotName").stringValue = _musicChoices[serializedObject.FindProperty("MsSnapshotIndex").intValue];
      
      EditorGUILayout.PropertyField(serializedObject.FindProperty("AtsSnapshotIndex"), new GUIContent("Atmos Snapshot"));
      serializedObject.FindProperty("AtsSnapshotName").stringValue = _atmosChoices[serializedObject.FindProperty("AtsSnapshotIndex").intValue];
      
      EditorGUILayout.PropertyField(serializedObject.FindProperty("EsSnapshotIndex"), new GUIContent("Enviro Snapshot"));
      serializedObject.FindProperty("EsSnapshotName").stringValue = _enviroChoices[serializedObject.FindProperty("EsSnapshotIndex").intValue];
      
      EditorGUILayout.PropertyField(serializedObject.FindProperty("AcsSnapshotIndex"), new GUIContent("Actor Snapshot"));
      serializedObject.FindProperty("AcsSnapshotName").stringValue = _actorChoices[serializedObject.FindProperty("AcsSnapshotIndex").intValue];
      
      EditorGUILayout.PropertyField(serializedObject.FindProperty("SsSnapshotIndex"), new GUIContent("Shade Snapshot"));
      serializedObject.FindProperty("SsSnapshotName").stringValue = _shadeChoices[serializedObject.FindProperty("SsSnapshotIndex").intValue];

      // --- AUDIO ---------------------------------------

      EditorGUILayout.PropertyField(serializedObject.FindProperty("transitionTime"), true);
      EditorGUILayout.PropertyField(serializedObject.FindProperty("manualMapTrigger"), true);

      //Save all changes made on the inspector
      serializedObject.ApplyModifiedProperties();
    }
  }
  ```

With that done, we can actually get to the scene content now, yay!

#### Unity Project Scene Content

So we will need a few objects for a basic Hollow Knight Scene. Most of these are just for organizing. Also, all have their Transform as:  
| Part     | X | Y | Z |
|----------|---|---|---|
| Position | 0 | 0 | 0 |
| Rotation | 0 | 0 | 0 |
| Scale    | 1 | 1 | 1 |

- GameObject `__Initializer`, to which we will add 3 MonoBehaviours
  - MonoBehaviour `PatchAreaTitleController`
    - Here you can leave most values as default
    - `Sub Area`: You can uncheck
    - `Area Event`: Write "MyFirstCustomSceneMod_AreaTitle"
      - This key with "_SUPER", "_MAIN" and "_SUB" will be used for language lookup of the area name
    - `Visited Bool`: Write "MyFirstCustomSceneMod_VisitedArea"
  - MonoBehaviour `SceneManagerPatcher`
    - Here you can leave most values as default
    - Gameplay Scene Settings
      - `Map Zone`: Can be adjusted depending on where in the world the custom Scene should be. In our case, `TOWN` for Dirtmouth.
      - `Environment Type`: Will change the particles when walking and landing.
      - `Darkness Level`: Will change how dark the Scene is.
      - `No Lantern`: Dis- or Enables the use of the Lumafly Lantern.
    - Audio Snapshots
      - `Atmos Cue Set`: Can be left as defaut.
      - `Atmos Cue Snapshot`: Can be left as defaut.
      - `Atmos Cue Is Channel Enabled` array: Can be left as defaut.
      - `Music Cue Set`: Can be left as default, but you can change these for custom background music.
      - `Music Cue Snapshot`: Can be left as default, but you can change these for custom background music.
      - `Music Cue Channel Info Clips` array: Can be left as default, but you can change these for custom background music.
      - `Music Cue Channel Info Syncs` array: Can be left as default, but you can change these for custom background music.
      - `Infected Music Cue Set`: Can be left as default, but you can change these for custom background music.
      - `Infected Music Cue Snapshot`: Can be left as default, but you can change these for custom background music.
      - `Infected Music Cue Channel Info Clips` array: Can be left as default, but you can change these for custom background music.
      - `Infected Music Cue Channel Info Syncs` array: Can be left as default, but you can change these for custom background music.
      - `Infected Music Cue Channel Info Syncs`: Can be left as default.
      - `Music Delay Time`: Can be left as default.
      - `Music Transition Time`: Can be left as default.
      - `Music Snapshot`: Can be left as default.
      - `Atmos Snapshot`: Can be left as default.
      - `Enviro Snapshot`: Can be left as default.
      - `Actor Snapshot`: Can be left as default.
      - `Shade Snapshot`: Can be left as default.
      - `Transition Time`: Can be left as default.
    - Mapping
      - `Manual Map Trigger`: Off to allow Benches in the Scene to trigger mapping. On to not allow that.
  - MonoBehaviour `PatchPlayMakerManager`
    - `Manager Transform`: Drag and drop the `_Managers` GameObject here.
- GameObject `_Areas`
- GameObject `_Camera Lock Zones`
  - GameObject `Main`, placed at (`15`, `8.5`, `0`), to which we will add 2 MonoBehaviours
  - MonoBehaviour `BoxCollider2D`
    - `Is Trigger`: Check it.
    - `Size`: Set to `32`x`19`.
  - MonoBehaviour `CameraLockArea`
    - `Camera X Min`: Set to `15`.
    - `Camera Y Min`: Set to `8.5`.
    - `Camera X Max`: Set to `15`.
    - `Camera Y Max`: Set to `8.5`.
    - `Prevent Look Up`: Check it.
    - `Prevent Look Down`: Check it.
    - `Max Priority`: Leave it unchecked.
- GameObject `_Effects`
- GameObject `_Enemies`
- GameObject `_Items`
- GameObject `_Managers`
- GameObject `_Markers`
- GameObject `_NPCs`
- GameObject `_Props`
- GameObject `_Scenery`, to which we will add 1 MonoBehaviours
  - MonoBehaviour `SpritePatcher`
    - `Shader`: Set to `Sprites/Lit`.
    - `Scale`: Set to `1`.
    - This allows any sprite added as a child of `_Scenery` to be actually viewed in game as not pink boxes.
    - Alternatively, put the MonoBehaviour on a child under which you then put the sprites.
- GameObject `_Transition Gates`
  - GameObject `left1`, to which we will add 2 MonoBehaviours
    - MonoBehaviour `BoxCollider2D`
      - `Is Trigger`: Check it.
      - `Size`: Set to `1`x`5`.
    - MonoBehaviour `TransitionPoint`
      - `Always Enter Left`: Check it.
      - `Target Scene`: Set to `Town`.
      - `Entry Point`: Set to `bot1`.
  - GameObject `right1`, to which we will add 2 MonoBehaviours
    - MonoBehaviour `BoxCollider2D`
      - `Is Trigger`: Check it.
      - `Size`: Set to `1`x`5`.
    - MonoBehaviour `TransitionPoint`
      - `Always Enter Right`: Check it.
      - `Target Scene`: Set to `Crossroads_01`.
      - `Entry Point`: Set to `top1`.
- GameObject `BlurPlane`, to which we will add 3 MonoBehaviours
  - MonoBehaviour `MeshFilter`
    - `Mesh`: Create a new Mesh in like Blender or something of a square, any flat plane is good and select it here.
  - MonoBehaviour `MeshRenderer`
    - `Materials`: You can add a custom material there.
    - Lighting
      - `Cast Shadows`: Off.
      - `Receive Shadows`: Off.
      - `Contribute Global Illumination`: Off.
    - Probes
      - `Light Probes`: Off.
      - `Reflection Probes`: Blend Probes.
      - `Anchor Override`: None.
    - Probes
      - `Motion Vectors`: Per Object Motion.
      - `Dynamic Occlusion`: On.
  - MonoBehaviour `BlurPlanePatcher`
    - This will replace the Material you put on the `MeshRenderer`, but you can still put one there for visualization of where the BlurPlane would need to be.
- GameObject `TileMap`, set the Tag to `TileMap`, to which we will add 1 MonoBehaviour
  - MonoBehaviour `PatchTileMap`
    - `Width`: Set to `30`.
    - `Height`: Set to `17`.
    - `Columns`: Set to `1`.
    - `Rows`: Set to `1`.
    - `Part Size X`: Set to `30`.
    - `Part Size Y`: Set to `17`.
    - `Physics Material 2D`: Select the `Physics Material 2D` you created here.
    - `Render Data`: Drag and Drop the `TileMap Render Data` GameObject here.
- GameObject `TileMap Render Data`
  - GameObject `Scenemap`, set the Layer to `Terrain` and to which we will add 1 MonoBehaviour
    - MonoBehaviour `SceneMapPatcher`
      - `Tex`: Set to any black square sprite.
    - GameObject `Chunk 0 0`, which will have the Layer already set to `Terrain` and to which we will add 3 MonoBehaviours
      - MonoBehaviour `MeshFilter`
        - `Mesh`: Select the `TutorialScene` mesh.
      - MonoBehaviour `MeshRenderer`
        - `Materials`: Add a custom material there.
        - Lighting
          - `Cast Shadows`: On.
          - `Receive Shadows`: On.
          - `Contribute Global Illumination`: Off.
        - Probes
          - `Light Probes`: Blend Probes.
          - `Reflection Probes`: Blend Probes.
          - `Anchor Override`: None.
        - Probes
          - `Motion Vectors`: Per Object Motion.
          - `Dynamic Occlusion`: On.
      - MonoBehaviour `Polygon Collider 2D`
        - `Material`: the `Physics Material 2D` we created.
        - `Is Trigger`: Off.
        - The shape can be either drawn manually or by right clicking the Bar of the `MeshFilter` MonoBehaviour and clicking "Create Collision"
