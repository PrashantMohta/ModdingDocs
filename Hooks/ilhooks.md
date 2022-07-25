---
title: IL Hooks
nav-order: 3
parent: Hooks
---
# IL Hooks

IL Hooks are a way to directly modify the code of a method. 

## When to Use IL Hooks

IL hooks can be an incredibly powerful tool to surgically modify small chunks of existing code. However, they
are not a swiss army knife, and are not appropriate to use in every situation. Using IL hooks is not a
reasonable replacement for any of the following:

* Replacing the functionality of a method by using an On hook without calling orig
  * It's really just more effort to use IL hooks.
* Editing an FSM in hopes to avoid compatibility issues
  * IL hooks are just as brittle as FSM edits, if not more.
  * If you're trying to modify the FSM in a major way, you can expect other mods to break regardless of what
    you do.
  * It's really just more effort to use IL hooks.

IL hooks are best used for small changes - changing magic numbers, altering the condition of an if statement,
or other tweaks where you would otherwise think to just copy-paste the original method and change a few 
lines.

## What is IL?

IL, short for Intermediate Language, is the lower-level language that .NET languages compile to. You may also
see it called CIL ([Common Intermediate Language](https://en.wikipedia.org/wiki/Common_Intermediate_Language),
its formal name) or MSIL (Microsoft Intermediate Language, its original name). It is an object-oriented,
stack-based bytecode language - if this makes no sense to you, that is extremely reasonable. Let's try to
make sense of it.

Object-oriented means that objects are first-class citizens. You may be familiar with this concept because
C# is also an object-oriented language. You can create classes with properties, fields, and methods, and then
create many instances of those classes.

Stack-based language means that the language operates on a stack. A stack is a programming concept that is useful
in many scenarios and, while initially confusing, can actually be quite easy to reason about. Consider a deck of 
playing cards on a table. You can put cards on top of the deck ("pushing" them onto the stack) or remove cards from 
the top of the stack ("popping" them from the stack). You can't easily remove cards from the bottom of the deck 
though. A stack-based language functions exactly the same way: instructions manipulate the stack by adding and
removing values. You start with an empty stack and end with an empty stack.

A bytecode language has a fixed set of instructions, and is not processor-dependent. This means, in theory,
that the program can run cross-platform, because they are not directly runnable and instead are executed by
a platform-specific virtual machine (Windows natively supports IL programs, on Mac and Linux you can run
them with Mono). Java is another example of a bytecode language. You can see a full list of CIL instructions
[here](https://en.wikipedia.org/wiki/List_of_CIL_instructions), but the most commonly used ones will be
covered here later.

As a simple example of IL, consider adding 2 numbers. In C#, that would look like this:

```csharp
int x = 2 + 3;
```

In IL, it would look like this:

```
ldc.i4 2 // push the constant value 2 onto the stack
ldc.i4 3 // push the constant value 3 onto the stack
add      // pop 2 values from the stack, add them, and push the result
stloc.0  // pop the value from the stack and store it in the 1st local variable (index 0)
```

## Reading IL Code in ILSpy/dnspy

To IL hook a method, first you'll need to find the IL code you'll be editing. Navigate to the code of 
interest in ILSpy or dnspy. Then, in the top bar, change from "C#" to "IL with C#." This will show you
the IL code, annotated with the equivalent C# code.

![Decompiling with IL](/ModdingDocs/Images/ILHooks/DecompileToIL.jpg)

Once you do, you'll see something like the following (this example is part of the Update method of the stag
cutscene's Monobehaviour).

![Stag example](/ModdingDocs/Images/ILHooks/DecompiledILExample.jpg)

## Fundamentals of IL Hooking

Creating an IL hook follows the same relatively simple formula regardless of what you're hooking.

1. **Attach** your hook to the method of interest.
1. **Match** instructions at or near where you want to modify the original code.
1. **Manipulate** the code by emitting and removing instructions.

Let's take a detailed look at the various ways to do each of these steps.

### Attach Hooks

Similar to `On` hooks, most methods in Hollow Knight can be hooked using a pattern like `IL.Class.Method += YourHook;`.
For example, if you wanted to change the way a given enemy gets counted in the Hunter's Journal, you would hook
`IL.PlayerData.CountJournalEntries`. IL hooks always have the signature `void(IlContext)`; your IDE can generate this
for you when you type `+=`, just like `On` hooks.

There are a few cases when it isn't this simple. To start with a simple real-world example, let's suppose you want
to make it so that the Hunter icon never shows up in the bottom corner of the screen. To do this, we would need
to hook into `EnemyDeathEffects.RecordKillForJournal`. When we take a look at the code in ILSpy, however, we see
something problematic:

```csharp
private void RecordKillForJournal()
{
    string killedBoolPlayerDataLookupKey = "killed" + playerDataName;
    string killCountIntPlayerDataLookupKey = "kills" + playerDataName;
    string newDataBoolPlayerDataLookupKey = "newData" + playerDataName;
    ModHooks.OnRecordKillForJournal(this, playerDataName, killedBoolPlayerDataLookupKey, killCountIntPlayerDataLookupKey, newDataBoolPlayerDataLookupKey);
    orig_RecordKillForJournal();
}
```

This isn't the original code! In fact, we can see that this is the caller for `ModHooks.OnRecordKillForJournal`. This
method has been patched by the modding API. The original code is actually in `orig_RecordKillForJournal`. We'll have to
make our changes there instead, but to do that, we'll need to make a custom IL hook first.

```csharp
// within your mod class, for this example

// find the method - in this case we know it's a private, non-static method on EnemyDeathEffects
private static MethodInfo origRecordKillForJournal = typeof(EnemyDeathEffects).GetMethod("orig_RecordKillForJournal",
    BindingFlags.NonPublic | BindingFlags.Instance);
// hold the actual hook
private ILHook ilRecordKillForJournal;

public override void Initialize() 
{
    ilRecordKillForJournal = new ILHook(origRecordKillForJournal, HideHunterIcon);
}

private void HideHunterIcon(ILContext il) { ... }
```

> Note: If you prefer not to write out the appropriate binding flags yourself (which is slightly more efficient at
runtime), you can also use `ReflectionHelper.GetMethodInfo(typeof(EnemyDeathEffects), "orig_RecordKillForJournal",
true);` to get the `MethodInfo` above. Throughout this documentation, the more complete, "vanilla" reflection approach
will be used, but you can usually take a similar approach with `ReflectionHelper`.

If you need to unhook the method (for instance, if your mod is toggleable), you can use `ILHook.Dispose()` to do so.

```csharp
public override void Unload()
{
    ilRecordKillForJournal.Dispose();
    ilRecordKillForJournal = null;
}
```

The other case where hooking is not straightforward is when hooking coroutines. Coroutines are IEnumerator "generator
methods" used by Unity to execute workloads over multiple frames. Typically these use `yield return` statements
to generate new values in the enumerator. To support this, the compiler actually generates a hidden, behind-the-scenes
class that implements your enumeration. Let's take for example the platforms in Queen's Gardens that drop out from
beneath you. The dropping is controlled by the `Flip` coroutine in the `DropPlatform` MonoBehaviour. Let's suppose we
want to change the time before the platform drops. We can't just hook `IL.DropPlatform.Flip`, even though that hook
exists, because the underlying IL code is just creating an instance of the generated class:

```
IL_0000: ldc.i4.0
IL_0001: newobj instance void DropPlatform/'<Flip>d__16'::.ctor(int32)
IL_0006: dup
IL_0007: ldarg.0
IL_0008: stfld class DropPlatform DropPlatform/'<Flip>d__16'::'<>4__this'
IL_000d: ret
```

Hopefully, this should make it clear that we actually need to modify code in the generated class (in this case,
`<Flip>d__16`). For coroutines, we need to modify the generated `MoveNext` method on the generated class. Similar to
above, we'd need to create a custom IL hook. To do this, we'd usually need to use reflection to get the generated type
and then get the method off the type. Fortunately, MonoMod provides a convenient extension method to do this work for 
us using compiler-generated attributes. To complete our example, here's the code needed to hook the `Flip` coroutine,
where SetDropTime is your defined IL manipulator.

```csharp
_hook = new ILHook
    (
        typeof(DropPlatform).GetMethod("Flip", BindingFlags.NonPublic | BindingFlags.Instance)
            .GetStateMachineTarget(),
        SetDropTime
    );
```

### Match Instructions

To work with IL, you need to create a cursor. In your IL manipulator:

```csharp
private void Manipulate(ILContext il)
{
    ILCursor cursor = new ILCursor(il).Goto(0);
    // ...
}
```

The `Goto(0)` is not strictly necessary in most cases, but is a good safety practice because you do always need to
call `Goto` or some variant of `GotoNext` before you can make any modifications.

Once you've made a cursor, the next step is to move the cursor to where you want to make your changes. Let's continue
the example of the drop platforms in QG (we'll keep using this example for the rest of the documentation). The IL code
that sets the delay looks like the following:

```text
// <>2__current = new WaitForSeconds(0.7f);
IL_0056: ldarg.0
IL_0057: ldc.r4 0.7
IL_005c: newobj instance void [UnityEngine.CoreModule]UnityEngine.WaitForSeconds::.ctor(float32)
IL_0061: stfld object DropPlatform/'<Flip>d__16'::'<>2__current'
```

We want to change the 0.7 to a different amount of delay. In order to do this, we'll have to move the cursor prior
to the ldc.r4 instruction.

```csharp
if (cursor.TryGotoNext
(
    i => i.MatchLdcR4(0.7f),
    i => i.MatchNewobj<WaitForSeconds>()
))
{
    ...
}
```

In `GotoNext` and `TryGotoNext`, we define a sequence of instruction predicates (i.e. functions that take in
`Instruction`s and return `bool`s). All the predicates must match sequential instructions in order. Most instructions
have a match predicate defined for you to use. In the example above, we can see we are trying to find an `ldc.r4`
with an operand of 0.7, and a `newobj` creating a `WaitForSeconds` (in this example, there's only one place where
`ldc.r4 0.7` happens, so you technically don't need the `newobj` check but it helps with readability).

The `Match*` predicates have several variants, depending on how much you care about the operand:
* If you want to match any `ldc.r4` and need to know the value, you can use `MatchLdcR4(out float value)`
* If you want to match any `ldc.r4` and don't care about what value, you can use `Match(OpCodes.Ldc_R4)`

These variants apply to other instructions as well. By default, `TryGotoNext` and `GotoNext` place the cursor before
the matched instructions, so the cursor would be at IL_0057 as shown below. You can change this by setting the first
argument to `MoveType.After` if needed, but in many cases it will be possible and more convenient to simply match a 
different set of instructions.

```text
// <>2__current = new WaitForSeconds(0.7f);
IL_0056: ldarg.0
>>>cursor is here<<< IL_0057: ldc.r4 0.7
IL_005c: newobj instance void [UnityEngine.CoreModule]UnityEngine.WaitForSeconds::.ctor(float32)
IL_0061: stfld object DropPlatform/'<Flip>d__16'::'<>2__current'
```

Depending on your preferred mode of failure, you can use `GotoNext` instead of `TryGotoNext`. In this case, you avoid
an if statement, but if you fail to match IL (i.e. your predicates are incorrect), you'll get an exception. In most
cases, you're doing a single modification in a known place matching a known pattern, so this may be preferred over
failing silently.

If you need to match several similar chunks of code, you can also use `TryGotoNext` in a while loop to make repeated 
modifications. For example, if you need to modify every change in velocity in `HeroController.FixedUpdate`, you can 
see there are several more than you'd want to hook individually. You can, however, see a pattern: every time the
velocity is changed, the exact same IL code is run. We can make the same modification to each appearance of this code
by doing the following (adapted from RandomGravityChange):

```csharp
public void ChangeVelocity(ILContext il)
{
    ILCursor cursor = new ILCursor(il).Goto(0);
        
    //Match all the times game goes to set velocity
    while (cursor.TryGotoNext
        (
            MoveType.Before,
            //happens when the vector is created. not matching this causes a while(true) to happen sometimes
            i => i.Match(OpCodes.Newobj),
            // the actual thing we want to adjust
            i => i.MatchCallvirt<Rigidbody2D>("set_velocity")
        ))
    {
        //because the newObj isnt the target of my IL hook, the set velocity is
        cursor.GotoNext();
        //want to take in current Vector2 on stack and replace it with my own
        cursor.EmitDelegate<Func<Vector2, Vector2>>(GetNewVelocity);
    }
}
```

### Manipulate Code

Once your cursor is positioned in the right place, you can begin to make modifications to the IL code. This is a good
time for a reminder that the stack starts empty and needs to end empty - in other words, you need to keep the stack
balanced. Any time you add a value, you should remove a value, any time you remove a value you should add a value.
With that in mind, let's look at some common ways to manipulate IL.

`cursor.Next` refers to the instruction the cursor is pointing to. In some cases, it's possible to do simple
manipulations just by replacing the operand. In the drop platform example, we could simply do 
`cursor.Next.Operand = NewValue;` inside the if block. Note that when using this approach, the type of the operand
must be appropriate for the corresponding opcode - in this case, we're modifying an `ldc.r4` instruction, so the
operand must be a float.

The next most common method of manipulation is to add and remove instructions. We can achieve this by using `Remove`
and `Emit`. `Remove` removes the instruction at the cursor; continuing from the previous example, removing a statement
would leave the IL in this state (note that this is bad; we've removed a value without adding a value so the stack is
unbalanced):

```text
// <>2__current = new WaitForSeconds(0.7f);
IL_0056: ldarg.0
>>>cursor is here<<< IL_005c: newobj instance void [UnityEngine.CoreModule]UnityEngine.WaitForSeconds::.ctor(float32)
IL_0061: stfld object DropPlatform/'<Flip>d__16'::'<>2__current'
```

We can then add back a new value using `Emit`, restoring the stack to a valid state.

```text
// <>2__current = new WaitForSeconds(0.7f);
IL_0056: ldarg.0
>>>cursor is here<<< ldc.r4 SomeValue // this would be the actual constant value
IL_005c: newobj instance void [UnityEngine.CoreModule]UnityEngine.WaitForSeconds::.ctor(float32)
IL_0061: stfld object DropPlatform/'<Flip>d__16'::'<>2__current'
```

Here's the full code to do the above manipulations:

```csharp
if (cursor.TryGotoNext
(
    i => i.MatchLdcR4(0.7f),
    i => i.MatchNewobj<WaitForSeconds>()
))
{
    cursor.Remove();
    cursor.Emit(OpCodes.Ldc_I4, SomeValue);
}
```

You can emit any kind of opcode, with or without an operand. As a reminder, there is a full list of opcodes
[here](https://en.wikipedia.org/wiki/List_of_CIL_instructions). You'll see most of the ones you'll usually use in
the process of investigating the IL code you're hooking. Covering the functionality is out of scope for this
documentation, but a few commonly used ones include:
* ldc.i4 (load constant 4-byte integer)
* ldc.r4 (load constant 4-byte float)
* ldstr (load a string constant)
* ldloc (load a local variable)
* ldarg (load a method argument)
* ldfld (load a field of a class)
* callvirt (call an instance method of an object)

> Note: many of these instructions have several shorthand variants, such as `ldc.i4.0`. These are functionally
identical to the normal opcode (such as `ldc.i4 0`), but use a different opcode and no operand. When matching,
the opcodes are not interchangable. The shorthand opcodes are more efficient and should be preferred when possible.

You'll notice that most of these are ways to add values to the stack. In fact, if you want to do meaningful processing
on the data, there is a much easier way to do that: by using `EmitDelegate`. This will allow you to emit an instruction
that will call your C# code. This is a common place for the stack to get imbalanced, because arguments to your methods
are removed from the stack. In general, this means you should do one of the following:
* Use `EmitDelegate<Action>` to add additional code that needs no inputs and has no outputs; for example, to add 
  logging or side effects.
* Use `EmitDelegate<Func<T, T>>` to add additional code that takes an input and replaces it with an output of the
  same type; for example, to apply some modifications to the original value.
* Add additional items to the stack, then use `EmitDelegate<Action<...>>` to add additional code that takes inputs
  and has no outputs; for example, to get values of local variables or arguments.
* Remove items from the stack, then use `EmitDelegate<Func<T>>` to add code that does complex processing and then
  adds a value to the stack; for example, to compute a value based on some condition rather than using a constant.

Of course, this is not a comprehensive list of all the possible ways you can handle inputs and outputs of delegates;
they are just a few rules of thumb. Just make sure to keep the stack balanced.

Let's see how to use `EmitDelegate` in action. Suppose we want to modify the drop platform time based on the player's
current health. Here is one way to accomplish that using `EmitDelegate`.

```csharp
if (cursor.TryGotoNext
(
    i => i.MatchLdcR4(0.7f),
    i => i.MatchNewobj<WaitForSeconds>()
))
{
    // move between the ldc.r4 and the newobj; we want to have the original time on the stack as an input
    cursor.GotoNext();
    cursor.EmitDelegate<Func<float, float>>(f => {
        return f + (0.5f * PlayerData.instance.GetInt(nameof(PlayerData.health)));
    });
}
```

> Note: In most cases, you should use a named function rather than a lambda when emitting a delegate. A lambda is used
here for brevity.


## Debugging Broken IL

IL hooks can be incredibly difficult to debug. This is because in an IL hook, if you make an error, you'll usually
end up with invalid code, and the compiler has no way to warn you since you've made all the modifications at runtime.
This manifests itself as an `InvalidProgramException`, which often provides very little context on what the error is.
In many cases, the best chance you have to fix broken IL is to get out a pen and paper and work your way through the
state of the stack as instructions run, until you get to the code you've modified. However, there are a few common
errors that are typically caused when IL hooking, and being able to identify them can speed up your debugging
significantly.

The most common error to see, as we've discussed several times in this documentation already, is an unbalanced stack.
This can be caused in several ways, most commonly pushing additional data onto the stack without consuming it
appropriately or adding a delegate with arguments that doesn't push back onto the stack. If there are too many values 
on the stack, you'll usually see an error message like "Invalid IL code in (the method you hooked): (some IL address): 
ret" or "Invalid IL code in (the method you hooked): (some IL address): (some instruction close after where you 
modified code)," depending on if you added or removed values from the stack respectively.

Another common cause of errors is putting invalid data type on the stack (for instance, loaded a float when you should
have loaded an int). This will look quite similar to the error above, where some code near where you made your 
modification will show as invalid. This is because that code is expecting a certain type of input still; in the case 
above you are providing no input, in this case you are providing the incorrect input.

A final, less common cause of errors is creating invalid branch instructions. This can happen when modifying code
after the end of an if block, or when trying to emit if/branch instructions in IL. Typically, the error is caused
when you branch into the middle of a series of operations, so the stack is not in an expected state. This will look
similar to the previous 2 errors, since the stack would be missing expected values. In some cases, this error can also
be caused by removing a label that is the target of a branch instruction (for example, if there is a `brtrue IL_066C`
somewhere and you delete the line at `IL_066C`). In this case, the error message will say something like "Label #X is
not marked in method."

## Additional Reading

In this documentation, you've been presented several partial examples. Many of these examples are adapted from a 
heavily annotated example by [Flib](https://github.com/flibber-hk). Here is the original/complete example for updating
the wait times for the QG drop platforms: https://gist.github.com/flibber-hk/47bd1e713405fa5936255dd2265c3bb3.
This is a good read as well, as it presents a more complete view of the code and is more example-oriented.
