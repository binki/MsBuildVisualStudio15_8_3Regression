This repository demonstrates a regression in Visual Studio 15.8.3.

# Issue

Starting in Visual Studio 15.8.3, references to item metadata in `<ItemDefinitionGroup/>` are not expanded when building with the Visual Studio GUI (CLI is unaffected).
For example, if you had something like the following, future expansion of `%(MyItem.MyDefinedMetadata)` would yield `%(Identity)` instead of the expanded value of the `Identity` metadata:

```xml
<ItemDefinitionGroup>
  <MyItem>
    <MyDefinedMetadata>%(Identity)</MyDefinedMetadata>
  </Myitem>
</ItemDefinitionGroup>
```

If you were to build this with Visual Studio 15.7.6, 15.8.2, or by running `MSBuild\15.0\Bin\amd64\MSBuild.exe` from Visual Studio 15.8.3’s installation folder, then the value is properly expanded.

# Demo

This project defines [items](Directory.Build.props) and [targets](Directory.Build.targets) which are construed to successfully build on unaffected versions of Visual Studio and fail to build on affected versions of Visual Studio.
The targets copy all `**/*.cs` files to the output directory.
When calling the `<Copy/>` Task, the `SourceFiles` parameter is set to metadata which is defined using `<ItemDefinitionGroup/>`.
The result is that affected versions of Visual Studio attempt to copy files named `%(RelativeDir)%(Filename)%(Extension)` to the destination.
Since I have not included any files by that name in this project, the copy fails.
However, on unaffected versions of Visual Studio, the expansion occurs correctly and the copy succeeds.

This demo is based on actual undisclosed code I use.
I would expect any project relying on `<ItemDefinitionGroup/>` to either be uncompilable or have subtly incorrect behavior when compiled using the GUI of an affected version of Visual Studio.
In the case of my undisclosed code, the result is a build failure prior to compilation happening.

# Repro Steps

1. Install Visual Studio 15.8.3 with C# project support.
2. Open the solution in Visual Studio.
3. Use the GUI to build the project (e.g., Build→Build Solution). You will experience a build error.
