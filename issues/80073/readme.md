This is to help with debugging GH issue: [.NET 7 application hangs during GC · Issue #80073 · dotnet/runtime (github.com)](https://github.com/dotnet/runtime/issues/80073). The problem is instead of creating an allocation context that's ~8k, we seem to have one that's almost as big as a whole region! 

### Explanation of the builds

The **baseline** build is built on the release/7.0 branch.

C:\runtime7.0\src>git log -n1
commit e516d184b0414e4d1b97e8645a1cf5f7a51596d5 (HEAD -> 7.0, upstream/release/7.0, release/7.0)
Author: github-actions[bot] <41898282+github-actions[bot]@users.noreply.github.com>
Date:   Fri Jan 6 09:27:40 2023 -0800

    [mono][aot] Fixed decompose_flag propagation, addresses #79710 (#79855)
    
    Co-authored-by: Jan Dupej <jandupej@microsoft.com>

It has the following change -

+ Build clrgc.dll with regions instead of segments (the shipped version of clrgc.dll uses segments which is from .net 6.0).

The **with_change** build is the *baseline build + the following change* -

+ Induces an AV when it detects an allocation context size that's too large for SOH (I used LOH threshold * 2) and records some info about the most recent allocation contexts for debugging. 

### How to test

clrgc.dll is located in the same directory as coreclr.dll. Please following the instructions below to test -

1. Save your current clrgc.dll somewhere, copy over the clrgc.dll from the **with_change** directory.
2. To use clrgc.dll, please set this env var **COMPlus_GCName** to clrgc.dll (depending on how you set this, you might need quotes around clrgc.dll, for example if you use powershell to set it). You can verify whether you are actually using clrgc.dll by looking at the process to see if clrgc.dll is loaded. If it's not loaded, it means the env var did not succeed.
4. Assuming you can still repro the original issue, please share a dump. 

I included the baseline build in case you cannot repro with the with_change build, you could run with the baseline build to see if the reason why you can't repro is because of the changes or simply because when you use the 7.0 GC in clrgc.dll (vs in coreclr.dll) that already made it not repro.

Thank you for helping us with debugging this issue!