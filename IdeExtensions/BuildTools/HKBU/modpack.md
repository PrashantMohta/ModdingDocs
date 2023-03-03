---
title: Mod Packer
nav_order: 3
parent: Hollow Knight Build Utils
---

Automatically pack the mod into a zip file and calculate the SHA256 value after the build.


## How to use

**Generally speaking, it can be used without any processing**, but if you need some special functions, you can continue reading.

### Add additional files
Use the `<OutputFiles Include="File Name"/>` item element.
Like this
```xml
<OutputFiles Include="Test1.txt" />
<OutputFiles Include="Library.dll" />
```

**Warning:**
- `File Name` uses a file name relative to `$(TargetDir)`, which means that for files to be added, you need to set `<CopyToOutputDirectory>Always</CopyToOutputDirectory>` or `<Private>true</Private>`

### Process packed files
Like this
```xml
<Target Name="[Target Name]" AfterTargets="PackOutputMod">
    <WriteLinesToFile File="$(ReleaseInfoPath)"
                      Overwrite="true"
                      Lines="SHA256: $(OutputZipSHA256)"
                      Encoding="UTF-8"
                      />
  </Target>
```

Available properties:
- `$(OutputZipSHA256)`: SHA256 of the output publish zip

## Configuration

- `$(ExportDir)`: Specifies the directory for the packaged output. Defaults to "$(ProjectDir)\bin\Publish\"
- `$(PublishZipPath)`: Specify the save path of the ZIP output after packaging. Defaults to "$(ExportDir)\Publish.zip"
- `$(ReleaseInfoPath)`: Specifies the output location of the ReleaseInfo information constructed after packaging. Currently, only SHA256 is included. Default is "\$([System.IO.Path]::GetDirectoryName('$(PublishZipPath)'))\ReleaseInfo.txt"

