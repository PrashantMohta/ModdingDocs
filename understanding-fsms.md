# PlaymakerFSMs
## Introduction:  
Playmaker is a visual state machine editor that Team Cherry use quite extensively across the game to control how things behave. Unfortunately this means that much of the game's behavior is not written in easy to work with code form, but is represented as a flow chart of states.

Playmaker documentation is available on [hutonggames' website](https://hutonggames.fogbugz.com/default.asp?W133).

## Viewing FSMs:  
FSMs from the game can be viewed using a tool developed by nesrak1 known as [FSMViewAvalonia](https://prashantmohta.github.io/ModdingDocs/#todo-section) 

## Modifying FSMs:  
While modifying FSMs directly via code is possible; it is such a common task most [Dependency Mods](https://prashantmohta.github.io/ModdingDocs/#todo-section) have a utility for working with FSMs. Most of them are similar, with some extra features added here and there. For the purposes of this guide we will use the `FsmUtil` from [Satchel](https://github.com/PrashantMohta/Satchel)

 > Note: be sure to add the `using` directives for `HutongGames.PlayMaker` , `HutongGames.PlayMaker.Actions` & `Satchel`

### Common Tasks with FSMs:  
 - [Access an FSM](#access-an-fsm)
 - [Working with States](#working-with-states)
 - [Working with Actions](#working-with-actions)
 - [Custom Action](#custom-action)
 - [Working with Transitions](#working-with-transitions)

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
