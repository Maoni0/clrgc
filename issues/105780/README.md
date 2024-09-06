# Custom GC for investigating 105780

This custom GC is meant for investigating the issue [#105780](https://github.com/dotnet/runtime/issues/105780)

In one dump, it is observed that the number of background GC thread is one less than the number heaps, causing a deadlock. We do not understand the cause yet.

Therefore we built this instrumented binary, using logging and assertions, we hope to get more information if the situation ever happen again.

The full source is available [here](https://github.com/cshung/runtime/tree/public/deadlock-instrumentation), and we can see the change [here](https://github.com/dotnet/runtime/compare/v8.0.7...cshung:runtime:public/deadlock-instrumentation).

At a high-level, the change added an array so that we can store a set of variable values at various time. The change also added a few assertions so that it crashes as soon as the faulty situation appears.

Thanks a lot for the help!