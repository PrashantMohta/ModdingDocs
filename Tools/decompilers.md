# IlSpy.
[ILSpy](https://github.com/icsharpcode/ILSpy) is an open-source .NET assembly browser and decompiler. It allows users to view and decompile binary files into C# or [IL](https://prashantmohta.github.io/ModdingDocs/Hooks/ilhooks.html#what-is-il) where it is human readable.

## Basic Search:
1. Open [ILSpy](https://github.com/icsharpcode/ILSpy).
2. Load your .NET assembly by navigating to **File > Open** or dragging and dropping the `.dll` or `.exe` file into the ILSpy window.
3. Use the **Search Bar** and type the name of a class, function, or field to locate it across all loaded assemblies.

## Analyse
The **Analyse** feature in ILSpy helps trace the usage of a specific field or function within the loaded assemblies.

### How to Use Analyse:  
1. Get the desired field/function in the decompiled code.
2. Right-click the item and select **Analyse**.
3. The **Analysis Panel** (on the right side of the application) will display information including usage of the field or function.

## TODO
- How to View IL Code
