# How to use a custom GC

Sometimes, we wanted to change the behavior of the GC. For example, to add logging so that we can debug issues. This document briefly describe how to use a custom GC.

## (Optional) build the custom GC

If you are given a branch, you can build the clrgc module yourself. [Here](https://github.com/cshung/runtime/tree/main/docs/workflow/building/coreclr) are the build instructions. Make sure you build it for the right architecture and flavor as needed.

Alternatively, you can ask the team to build one for you. It is advisable that you use clrgc module only from trusted sources.

## Upload the custom GC

Once you have the custom GC available, you can copy it to the machine that you need it. You can place the binary anywhere, but make sure the location is secured from potential malicious tampering.

## Figure out the base directory (required for .net 8 or below only)

You need to know where is the `coreclr` module is actually used in your process. In case of self-contained deployment, it is right next to the executable, otherwise, it is usually stored in the global installation directory, for example, on Windows, it will be something like this:

`C:\Program Files\dotnet\shared\Microsoft.NETCore.App\8.0.8\coreclr.dll`

If you have a dump, you can inspect the dump to check where the module is loaded. Other tools like SysInternals process explorer on windows can do that in a live process too.

## Configure the runtime

For .NET 9 or above, you can set use the GCPath configuration described [here](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector#path). As described in the doc, the configuration should be a full path to the clrgc module.

Otherwise, you will need to use the GCName configuration described [here](https://learn.microsoft.com/en-us/dotnet/core/runtime-config/garbage-collector#name). As described in the doc, the configuration should be a relative path from the coreclr module to the clrgc module.

In both cases, if you could change the configuration file, use the configuration file is easiest. This will make sure you impact exactly only the processes that have the config file changed. Alternatively, you can use the environment variable option as well, be careful that any subprocesses that get launched will also be impacted.

## Making sure the custom GC is used

If you could look at the module list (using a dump or otherwise), you can check that the clrgc module is loaded.

Alternatively, if it is easier, we can check the result of the `GC.GetConfigurationVariable()` API. One of the entries in the API should be `GCName:clrgcexp.dll` or something like that depending how you name the module and platforms.

## Troubleshooting

If the configuration is not done correctly, you might see something like this:

```txt
...
END: coreclr_initialize failed - Error: 0x8007007e
```

This means the runtime is unable to load the custom GC, you might want to check the paths again to make sure it is correct.