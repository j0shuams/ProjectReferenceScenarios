# ProjectReferenceScenarios

This repo contains one solution - Scenarios.sln. The solution is an example of how a C#/WinRT component is authored and consumed by projects via ProjectReference.
CsWinRTComponent uses CsWinRT version **1.1.4-prerelease.210312.6**.

There are two consuming projects: CppWinRT_WindowsConsoleApp and Cs_ConsoleApp. Both have a project reference to CsWinRTComponent. 

Because of how the targetes are configured in the version **1.1.4-prerelease.210312.6**, the project reference for the C++/WinRT app works as we expect.
This means the WinMD made by the CsWinRTComponent flows through the project reference and gets picked up by C++/WinRT. 
If the WinMD had not flown properly, the projection (CsWinRTComponent.h) would not have been created. 
Since we are flowing the WinMD properly, the C# app will fail because of NETSDK1130. 

To invert this behavior, you can change one line of the targets file that is a part of version **1.1.4-prerelease.210312.6**. 
To do this, go to your global nuget packge directory for this targets file -- usually  `C:\users\username\.nuget\microsoft.windows.cswinrt\1.1.4-prerelease.210312.6\build`

Inside `Microsoft.Windows.CsWinRT.Authoring.targets`, under the `GetTargetPath` target, around line 69, there is a tag of metadata added: 
`ResolveableAssembly`. Comment out this additional metadata and save the file. 
When you rebuild the solution, the C++/WinRT app will fail because it didn't make the projection, but the C# app will succeed. 

`ResolveableAssembly` being set to false makes it so that the WinMD is not a part of the item group, used by MSBuild, that corresponds to project reference outputs.
This means that dotnet will not see the WinMD, and thus no NETSDK1130 error. 
This also means that the C++ app will not see the WinMD, as it is not in the item group used to generate projections (in C++/WinRT's targets). 
