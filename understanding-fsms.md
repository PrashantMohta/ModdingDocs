---
nav_order: 12
---
# PlaymakerFSMs
## Introduction:  
Playmaker is a visual state machine editor that Team Cherry use quite extensively across the game to control how things behave. Unfortunately this means that much of the game's behavior is not written in easy to work with code form, but is represented as a flow chart of states.

Playmaker documentation is available on [hutonggames' website](https://hutonggames.fogbugz.com/default.asp?W133).
An explantions of Playmaker FSMs and how to view them is also available on the [radiance.host](https://radiance.host/apidocs/PlayMakerFSM.html) website

## Viewing FSMs:  
FSMs from the game can be viewed using a tool developed by nesrak1 known as [FSMViewAvalonia](Tools/fsmviewer.md) 

## Modifying FSMs:  
While modifying FSMs directly via code is possible; it is such a common task most [Dependency Mods](dependencymods.md) have a utility for working with FSMs. Most of them are similar, with some extra features added here and there. For the purposes of this guide we will use the `FsmUtil` from [Satchel](https://github.com/PrashantMohta/Satchel)

 > Note: be sure to add the `using` directives for `HutongGames.PlayMaker` , `HutongGames.PlayMaker.Actions` & `Satchel`

### Common Tasks with FSMs:  
 - [Access an FSM](#access-an-fsm)
 - [Working with States](#working-with-states)
 - [Working with Actions](#working-with-actions)
 - [Custom Action](#custom-action)
 - [Working with Transitions](#working-with-transitions)
 - [Creating custom FSM Actions](#creating-custom-fsm-actions)

### Access an FSM:  
To get Access to an FSM you need to know two things about it, the GameObject it is attached to and the `FsmName` of the FSM. This information can be obtained by [logging](logging.md) or using [FSMViewAvalonia](https://prashantmohta.github.io/ModdingDocs/#todo-section). 

Once you have access to both of them, gaining access to the fsm is as simple as doing :
```cs
var fsm = gameObject.LocateMyFSM("FsmName");
```
### Working with States:  
When working with states, the following methods allow you to add new states and access a existing states. 
Adding a new state using a string:  
```cs
var myFsmState = fsm.AddState("stateName"); 
```
Adding a new state using a FsmState object:
```cs
fsm.AddState(myFsmState);  
```
Accessing a state:
```cs
var state = fsm.GetState("stateName");
```

### Working with Actions:  
To specify an action that you need to get from a specific Fsm you need to know the name of the State and the index of that Action.

To get first action from state "stateName" in fsm:
```cs
var action = fsm.GetAction("stateName",0); 
``` 
To add action at the end of the existing actions:
```cs
fsm.AddAction("stateName",action); 
``` 
To add action at the index = 1:
```cs
fsm.InsertAction("stateName",action,1); 
``` 
To remove action from index 0:
```cs
fsm.RemoveAction("stateName", 0)  
``` 

> Note: After removing an Action from a position all positions greater than that will shift down by 1. For this reason it helps to use `removeActions` in a descending order of index.

### Custom Action:  
A common modding use case is to require a function be executed when an FSM is in a particular state, for such a scenario it is possible to use the following methods to insert an action at the appropriate position:

To add a custom action at the end of the actions list:  
```cs
fsm.AddCustomAction("stateName", ()=>{Log("Custom Action!")});
```
To add a custom action at the index = 2:  
```cs
fsm.InsertCustomAction("stateName", ()=>{Log("Custom Action!")},2);  
```

### Working with Transitions:  

To add a new transition from state "fromStateName" to state "toStateName" on event "onEventName":  
```cs
fsm.AddTransition( "fromStateName", "onEventName", "toStateName");
```
To remove the existing transition from state "fromStateName" on event "onEventName":  
```cs
fsm.RemoveTransition("fromStateName","onEventName")
```
To change the existing transition from state "fromStateName" on event "onEventName":  
```cs
fsm.ChangeTransition("fromStateName","onEventName","toStateName")
```

### Creating Custom FSM Actions
If you want to do something specific to an FSM but there isn't an action that already does that, you can create your own pretty trivially.
For this example I would like to conditionally flip the isTrue and isFalse events in this fsm.  
![An action from the "Direction Wall" state in Knight-Superdash](/ModdingDocs/Images/customfsmstateexample.jpg).     
The easiest way to do this would be to create a "ConditionallyFlip_BoolTest" Action.  
To get started, open [ILspy](Tools/decompilers.md) and search for the original Action (BoolTest in this case). Other than the attributes above the
fields, the code is copy paste friendly which can be used as a reference when creating the custom FSM action.  
The first step is create a new class and have it inherit from `FsmStateAction`. Then create the necessary variables to do the action. In our case it would look something like this
```cs
public class ConditionallyFlip_BoolTest : FsmStateAction
{
    public FsmBool boolVariable;
    public FsmEvent isTrue;
    public FsmEvent isFalse;
    public Func<bool> conditionToFlip;
}
```
We used the same variables and just added another `Func<bool>` which will help us decide when to flip. Optionally, create a constructor to make it easier to call
```cs
public ConditionallyFlip_BoolTest(FsmBool _boolVariable, string _isTrue, string _isFalse, Func<bool> _condition)
{
    boolVariable = _boolVariable;
    isTrue = FsmEvent.GetFsmEvent(_isTrue); 
    isFalse = FsmEvent.GetFsmEvent(_isFalse);
    conditionToFlip = _condition;
}
```
> Note: We passed in strings for the `isTrue` and `isFalse` Events because it is easier to get the events in the FSMAction  

The next step would be to create an `OnEnter` override. This is the code that will be run when the action is reached in the FSM. 
For this we can use the original code to aid us.
```cs
public override void OnEnter()
{
    //call the condition supplied to see if we should flip or not
    bool result = conditionToFlip();

    //if the results said that we shouldn't flip 
    if (result == false)
    {
        //do the same as original code (taken from dsnpy)
        Fsm.Event(boolVariable.Value ? isTrue : isFalse);
    }
    else
    {
        // else flip it
        Fsm.Event(boolVariable.Value ? isFalse : isTrue);
    }
    
    // Since what we wanted to do with the FSM is over, we call base.Finish();
    base.Finish();
}
```
If we dont want to do anything else with the variables in the action, the last step is to call `base.Finish()`. And with this we created our custom FSM Action.
To call this we would do: 
```cs
fsm.InsertAction(
    "Direction Wall", //our state
    new ConditionallyFlip_BoolTest(
        fsm.FsmVariables.FindFsmBool("Facing Right"), //the FSM bool
        "LEFT", //isTrue event name
        "RIGHT", //isFalse event name
        () => settings.ShouldFlip)); //the condition.
```
Notice that a `Func<bool>` was used rather than just bool because bools are passed by value and wouldn't get updated based on the current state of the variable.
> Note: In BoolTest there is a bool called `everyFrame` which would run the check every frame. We excluded this for explanation purposes but it can definitely be added.

### TODO
- when and how to access FSMs 
  - if object already exist, use GameObject.Find().LocateMyFSM
  - if you have reference of GameObject use go.LocateMyFSM, 
  - if you want general use On.PlayMakerFSM.Awake
- how to edit actions 
- how to use fsm variables 
- explanation of resources.assets and how to access
- how to take the go-fsmName from fsm viewer and get it into code 

