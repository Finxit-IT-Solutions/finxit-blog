---
title: 'The new XML based .NET solution file'
date: '2025-03-04'
draft: false
---

Over are the days of the old [.NET solution file format](https://learn.microsoft.com/en-us/visualstudio/extensibility/internals/solution-dot-sln-file?view=vs-2022)!

Why you ask? Because it looks like a _mess_:

```
Microsoft Visual Studio Solution File, Format Version 12.00
# Visual Studio Version 17
VisualStudioVersion = 17.13.35825.156
MinimumVisualStudioVersion = 10.0.40219.1
Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "ConsoleApp1", "ConsoleApp1\ConsoleApp1.csproj", "{B57133BE-05A9-40F6-9E8B-A9640044E781}"
EndProject
Global
	GlobalSection(SolutionConfigurationPlatforms) = preSolution
		Debug|Any CPU = Debug|Any CPU
		Release|Any CPU = Release|Any CPU
	EndGlobalSection
	GlobalSection(ProjectConfigurationPlatforms) = postSolution
		{B57133BE-05A9-40F6-9E8B-A9640044E781}.Debug|Any CPU.ActiveCfg = Debug|Any CPU
		{B57133BE-05A9-40F6-9E8B-A9640044E781}.Debug|Any CPU.Build.0 = Debug|Any CPU
		{B57133BE-05A9-40F6-9E8B-A9640044E781}.Release|Any CPU.ActiveCfg = Release|Any CPU
		{B57133BE-05A9-40F6-9E8B-A9640044E781}.Release|Any CPU.Build.0 = Release|Any CPU
	EndGlobalSection
	GlobalSection(SolutionProperties) = preSolution
		HideSolutionNode = FALSE
	EndGlobalSection
	GlobalSection(ExtensibilityGlobals) = postSolution
		SolutionGuid = {AE6B485E-2B0A-4FA2-B30F-925B18053BC2}
	EndGlobalSection
EndGlobal
```

I'm not even sure, what kind of markup language this is supposed to be. I guess, plain text?

## The XML comeback in 2024/**2025**

The year is 2024/2025, everything is entirely occupied by `JSON` or `YAML`. Well, not entirely... One small company of indomitable XML lovers still holds out against the Cloud Native invaders.

Okay, enough [Asterix](https://en.wikipedia.org/wiki/Asterix) jokes. It actually makes sense, as Microsoft abandoned the `project.json` file back in 2016, that was introduced with the _brand new_ .NET Core. This won't be a debate about a decision from 2016 though. Let history be history.

I think it makes sense that Microsoft is also using XML for its new `slnx` solution file to be coherent with the `csproj` file.

## The new slnx format

The new `slnx` file follows the dogma of the `csproj` format. Convention over configuration. Simple. `XML`. I think it is awesome!

See for yourself:
```xml
<Solution>
  <Project Path="ConsoleApp1\ConsoleApp1.csproj" Type="Classic C#" />
</Solution>
```
## Prerequisites

First, you need at least version `9.0.200` of the .NET SDK. It isn't specifically mentioned in any release notes yet, but it was part of the [`9.0.200-preview` release notes](https://github.com/dotnet/core/blob/main/release-notes/9.0/9.0.0/9.0.200-preview.md). And that's it, the `dotnet` command now supports `slnx` solution files.

What's that, you want support in your favourite IDE? Oh okay.

### Visual Studio

At the time of writing this, the support for `slnx` files in Visual Studio is hidden behind a preview toggle.\
You can enable support under `Tools` → `Options` → `Environment` → `Preview Features` → `Use Solution File Persistence Model`.

Thankfully the description is more helpful than the title...
![Visual Studio Option to enable slnx support](/assets/posts/slnx-visual-studio-option.png)

### Rider

I quickly opened a `slnx` file and everything worked fine. It seems to be supported since version `2024.2`, as mentioned in their [release notes](https://www.jetbrains.com/rider/whatsnew/2024-2/#key-updates).

## Migration

### CLI

This one is already in the [official documentation](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-sln#migrate), you can simply migrate with `dotnet sln [<SOLUTION_FILE>] migrate`.

### Visual Studio

Unfortunately, Visual Studio doesn't have a simple right click migrate option, as like for similar migrations in the past.

The way to go: Select the solution file in the Solution Explorer, then `File` → `Save <Solution>.sln as` → Select `XML Solution (*.slnx)` as `Save as type`.

![Save <Solution>.sln as](/assets/posts/slnx-visual-studio-1.png)
![XML Solution (*.slnx)](/assets/posts/slnx-visual-studio-2.png)

After that close the current solution and open the new `slnx` file.

### Rider

Right click on the solution file → `Save as...` → `Save as XML solution (.slnx)`

![Save as XML solution (.slnx)](/assets/posts/slnx-rider.png)

Rider will then automatically ask you, if you want to load that new solution.

## Conclusion & Future

I love the new format. It is simple, it is readable. Just a very similar improvement as with the old `csproj` and the new one. And it will probably be the new default very soon.

James Montemagno, a Principal Manager at Microsoft mentioned as a comment on the [.NET February 2025 servicing releases updates blog post](https://devblogs.microsoft.com/dotnet/dotnet-and-dotnet-framework-february-2025-servicing-updates/), that there is more to come for the `.slnx` format in the future:
![James Montemagno, a Principal Manager at Microsoft mentioned there is more to come](/assets/posts/slnx-microsoft-blog.png)
