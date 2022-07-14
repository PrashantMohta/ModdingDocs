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
```msil
ldc.i4 2 // push the constant value 2 onto the stack
ldc.i4 3 // push the constant value 3 onto the stack
add      // pop 2 values from the stack, add them, and push the result
stloc.0  // pop the value from the stack and store it in the 0th local variable
```

## Reading IL Code in ILSpy/dnspy

To IL hook a method, first you'll need to find the IL code you'll be editing. Navigate to the code of 
interest in ILSpy or dnspy. Then, in the top bar, change from "C#" to "IL with C#." This will show you
the IL code, annotated with the equivalent C# code.

![Decompiling with IL](/Images/ILHooks/DecompileToIL.png)

Once you do, you'll see something like the following (this example is part of the Update method of the stag
cutscene's Monobehaviour).

![Stag example](/Images/ILHooks/DecompiledILExample.png)

### TODO
- reading IL code from ILspy
- explanation of matching, how to match
- list common OpCodes and what they do
- Explain GotoNext(); GotoNext(params bool predicate); TryGoToNext(params bool predicate);
- Explain Emit, EmitDelegate
- Explain Remove();
- Explain common patterns for replacing
  - cursor.EmitDelegate<Func<float,float>>
  - cursor.Remove(); cursor.Emit(OpCodes, val)/ cursor.EmitDelegate(() => val)
  - cursor.Next.Operand = newval
- understand common errors such as
    - invalid stack
        - when stack has a value thats not used
        - when stack has a value thats not expected (int instead of string)
        - when stack has missing value
    - invalid label
        - when remove is used to remove an instruction that is used to transfer control (in if statement)
- While vs If in matching
- Ienumarator IL hooking and how to do it
- - how to run your code mid function (will probably not be able to change value of parameters of functions tho)
