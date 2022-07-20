# IL Hooks

IL Hooks are a way to directly modify the code of a method. 

## When to Use IL Hooks

IL hooks can be an incredibly powerful tool to surgically modify small chunks of existing code. However, they
are not a swiss army knife, and are not appropriate to use in every situation. Using IL hooks is not a
reasonable replacement for any of the following:

* Replacing the functionality of a method by using an On hook without calling orig
  * It's really just more effort to use IL hooks
* Editing an FSM in hopes to avoid compatibility issues
  * IL hooks are just as brittle
  * If you're trying to modify the FSM in a major way, you can expect other mods to break regardless of what
    you do
  * It's really just more effort to use IL hooks

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

Stack-based language means that the language operates on a stack. Stacks are quite intuitive to think about.
Consider a deck of playing cards on a table. You can put cards on top of the deck ("pushing" them onto the
stack) or remove cards from the top of the stack ("popping" them from the stack). You can't easily remove 
cards from the bottom of the deck though. A stack-based language functions exactly the same way: 
instructions manipulate the stack by adding and removing values. You start with an empty stack and end 
with an empty stack.

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
    ilRecordKillForJournal = new ILHook(ilRecordKillForJournal, HideHunterIcon);
}

private void HideHunterIcon(ILContext il) { ... }
```

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

These variants apply to other instructions as well.

Depending on your preferred mode of failure, you can use `GotoNext` instead of `TryGotoNext`. In this case, you avoid
an if statement, but if you fail to match IL (i.e. your predicates are incorrect), you'll get an exception. In most
cases, you're doing a single modification in a known place matching a known pattern, so this may be preferred over
failing silently. If you need to match several similar chunks of code, you can also use `TryGotoNext` in a while loop
to make repeated modifications.

### Manipulate Code

explain remove
explain emit
explain emitdelegate

## Debugging Broken IL

explain why IL hooks are difficult to debug and why they produce InvalidProgramException
explain common causes of errors and how to identify them
 - stack is unbalanced
 - wrong type of data
 - invalid label
