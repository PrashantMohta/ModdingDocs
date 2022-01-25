# IL Hooks

### TODO
- reading IL code from Dnspy
- explanation of matching, how to match
- list common OpCodes and what they do
- Explain GotoNext(); GotoNext(params bool predicate); TryGoToNext(params bool predicate);
- Explain Emit, EmitDelegate
- Explain Remove();
- How to replace value in a function
- how to "insert code" in a function
- understand common errors such as
    - invalid stack
        - when stack has a value thats not used
        - when stack has a value thats not expected (int instead of string)
        - when stack has missing value
    - invalid label
        - when remove is used to remove an instruction that is used to transfer control (in if statement)
- While vs If in matching
- Ienumarator IL hooking and how to do it
