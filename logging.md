# Logging
Mods currently don't have a way to attach a debugger to the game. So the best way to debug your code is to add logs to it. When initially developping the mod it is good to add logs to make sure the sequence of code execution is as you expect, and the values supplied to your code is what you expect. 
## Where Can I see Logs?
There are 2 ways to see your logs:

 1. By [locating your modlog.txt](#finding-your-modlog) in the saves folder (see below), which contains all the logs of the current game session. This can be requested from the user of your mods if they have encountered some unexpected results.
> To access previous sessions modlogs, open the `Old ModLogs` folder in the saves folder.
2. By enabling the in-game console so view logs in-game in real time.
To do this, in the saves folder open `ModdingApi.GlobalSettings.json` and setting `ShowDebugLogInGame` to true.
```txt 
"ShowDebugLogInGame": true,
```

## Creating a Log:
To create a log in your main class (the one that inherts from `Mod`, it  as as easy as calling the log functions 
```cs 
public class MyFirstMod : Mod
{
	public override void Initialize()
	{
		Log("Initialize is running.");
	}
}
```
This will create the line (in the modlog and/or ingame console
```txt 
[MyFirstMod] - Initialize is running.
```

However in your other classes you will need to create a static variable of your main class and using that.
Example:
```cs 
public class MyFirstMod : Mod
{
	//create a public static variable of your main class that can be accessed from other classes
	public static MyFirstMod Instance;
	
	public override void Initialize()
	{
		//Assign the variable named 'Instance' to the current instance of your main class using the keyword 'this'
		Instance = this;
		//Now the variable 'Instance' is useable in other classes to provide functionality that the main class has such as Logging.
	}
}

//create a MonoBehaviour so I can attach  it to a gameobject to run some code on it
public class MyMonoBehaviour: MonoBehaviour
{
	public void Start()
	{
		MyFirstMod.Instance.Log("My MonoBehaviour has been created.");
		//Other code
	}
}
```
Output:
```txt 
[MyFirstMod] - My MonoBehaviour has been created.
```
> Note: You can use the use `Modding.Logger` from the Modding API but it is not recommended to do so as it doesnt display the name of your mod in the log.

## Log Levels:  
There are 5 different types of logs:  
|Level| Function |Description|  
  
|-|-|-|  
|Error| `LogError();`| Should be used to log errors that occured when code is running.|  
|Warn| `LogWarn();`| Should be used to log warning to users.|  
|Info| `Log();`| Should be used for normal logs.|  
|Debug| `LogDebug();`| Should be used for log to debug code. Using default settings, it won't be seen on regular user's modlog|  
|Fine|`LogFine();`| Used by Modding API to log. Could be used but is bloated by logs from Modding API.|  
  
By default, `LogDebug` and `LogFine` will not be seen in the modlog and/or ingame console, To change the level of logs you can see, locate the `ModdingApi.GlobalSettings.json` in the saves folder. There you will be able to see a setting called `LoggingLevel`. the default is 2 but it is recommeded to set it to 1:
```txt 
"LoggingLevel": 1,
```

The acceptable levels for this range from 0-6 where 5 is the least and 0 is the most:  
|Level|Amount of logging|  
  
|--|--|  
|5|None|  
|4|Error.|  
|3|Error, Warn.|  
|2|Error, Warn, Info. (This is the default setting)|   
|1| Error, Warn, Info, Debug.|   
|0| Error, Warn, Info, Debug, Fine.|   
   
## Player.log:  
Some unity errors are not logged to `modlog.txt` but rather to `player.log`. This can be found in the folder as `modlog.txt`. It is very useful especially because it provides the stack traces of the errors.
If you want this can also be logged to by using `UnityEngine.Debug.Log();`

## Finding your modlog:  
`ModLog.txt` can be found in the same folder as your save files:
- Windows: `%APPDATA%\..\LocalLow\Team Cherry\Hollow Knight\`
- MacOS  : `~/Library/Application Support/unity.Team Cherry.Hollow Knight/`
- Linux  : `~/.config/unity3d/Team Cherry/Hollow Knight/`

To easily get ModLog on Windows:
- Press Windows logo key + R
- Copy and paste this into it `%appdata%\..\LocalLow\Team Cherry\Hollow Knight` and hit "OK"
- ModLog.txt is in this folder.
