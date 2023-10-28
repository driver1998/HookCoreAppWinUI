# HookCoreAppWinUI

This is a hacky hook to allow WinUI 3 to run in UWP environment.

## Disclaimer

Windows App SDK and WinUI 3 is **NOT SUPPORTED** in UWP, this is just a fun science project expoliting an old workaround method, you should **NEVER EVER USE IT IN PRODUCTION ENVIRONMENT**.

Also this is my first ever API hook, it might contain some bone-headed mistakes, crash your system or destroy your data, **USE IT AT YOUR OWN RISK**. While you are at it, review the code, it isn't long ;-).

## How to use

Windows App SDK version 1.4.2 is tested.

You'll need to write your own entrypoint (`main` method). As the auto-generated one will not work in UWP.

Just load this dll before `Microsoft.UI.Xaml.Application.Start`, like this:

```csharp
public static class Program
{
    [DllImport("Microsoft.ui.xaml.dll")]
    private static extern void XamlCheckProcessRequirements();

    [DllImport("api-ms-win-core-libraryloader-l2-1-0.dll", SetLastError = true)]
    private static extern IntPtr LoadPackagedLibrary([MarshalAs(UnmanagedType.LPWStr)] string libraryName, int reserved = 0);

    [global::System.MTAThread]
    static void Main(string[] args)
    {
        // Enable the hook here
        LoadPackagedLibrary("HookCoreAppWinUI.dll");

        XamlCheckProcessRequirements();

        // Do your normal WinUI 3 initialization
        WinRT.ComWrappersSupport.InitializeComWrappers();
        Microsoft.UI.Xaml.Application.Start(
            // ...
        );
    }
}
```

While we are at it, define `DISABLE_XAML_GENERATED_MAIN` in the project file so it won't generate its own `main`:
```xml
<Project Sdk="Microsoft.NET.Sdk">
    <Sdk Name="Microsoft.Build.CentralPackageVersions" Version="2.0.1" />
    <PropertyGroup>
        <TargetFramework>$(SamplesTargetFrameworkMoniker)</TargetFramework>
        <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
        <OutputType>WinExe</OutputType>
        <UseWinUI>true</UseWinUI>
        <!-- Disable auto generated entrypoint -->
        <DefineConstants>DISABLE_XAML_GENERATED_MAIN</DefineConstants>
    </PropertyGroup>
</Project>
```

Lastly, update `Package.appxmanifest` and use a fixed entrypoint (if you keep it as is it will be `Windows.FullTrustApplication` which means an full-trust Win32 app).

It should be your `App` class.

```xml
  <Applications>
    <Application Id="App" Executable="$targetnametoken$.exe" EntryPoint="AppUIBasics.App">
    </Application>
  </Applications>
```

Also remove the `runFullTrust` capability, and add whatever you need.

Now you should build, deploy and run your WinUI 3 UWP.