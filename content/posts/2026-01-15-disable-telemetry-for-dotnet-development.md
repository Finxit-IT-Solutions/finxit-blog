---
title: 'Disable telemetry for .NET Development'
date: '2026-01-15'
draft: false
---

Did you know that Microsoft is collecting a lot of data about your usage of the `dotnet` CLI? Well, it probably does not come as a surprise. Also it is not a secret, [Microsoft is quite open about it](https://learn.microsoft.com/en-us/dotnet/core/tools/telemetry) and I assume most are aware of it. In fact, the `dotnet` CLI even tells you, when you first run it:
```
The .NET tools collect usage data in order to help us improve your experience. The data is collected by Microsoft and shared with the community. You can opt-out of telemetry by setting the DOTNET_CLI_TELEMETRY_OPTOUT environment variable to '1' or 'true' using your favorite shell.

Read more about .NET CLI Tools telemetry: https://aka.ms/dotnet-cli-telemetry
```

However, did you know that Microsoft is collecting a lot of data about your usage of the new Microsoft Testing Platform when you use third party libraries like [TUnit](https://tunit.dev/) or [xUnit](https://xunit.net/)? Probably not. Who would expect Microsoft data collection via third party libraries? But that's exactly what is happening now.

For TUnit, it was added a couple of weeks ago by a Microsoft Engineer and TUnit simply accepted the change. Besides a small note for the [1.7.16 release](https://github.com/thomhurst/TUnit/releases/tag/v1.7.16) of "*Add telemetry package by @Youssef1313 in [#4200](https://github.com/thomhurst/TUnit/pull/4200)*", ~there is no more information from TUnit regarding the impact of this change, or how to opt out of it.~ EDIT: after opening an issue with TUnit, they now since added a disclaimer in the release notes and [some documentation too](https://github.com/thomhurst/TUnit/pull/4429).

For xUnit, it was added with the November release 3.2.0. They at least were decent enough to put a [disclaimer in the release notes](https://github.com/xunit/xunit.net/blob/6966c9d94feb2c1565bc056a532df81b70ae5996/site/releases/v3/3.2.0.md) and added information including how to opt out in their [documentation](https://xunit.net/docs/getting-started/v3/microsoft-testing-platform.html#microsoft-testing-platform-telemetry).

I did not check for more libraries, but other testing frameworks are probably affected too.

While generally collecting anonymised data might not be a problem, there are variety of reasons why you might want to opt out of it. Let it be the simple desire for privacy, the work on more confidential projects or even the geopolitical issues we face nowadays and you to want to [stick it to the man](https://www.youtube.com/watch?v=AEN4Lp_prkc).

## Goodbye big brother

If you want to disable the telemetry for just the new Microsoft Testing Platform, it is enough to set `TESTINGPLATFORM_TELEMETRY_OPTOUT` to `1`.

If you want to disable all the .NET telemetry, set `DOTNET_CLI_TELEMETRY_OPTOUT` to `1`.

As it heavily depends on the operating system and the used shell, I'll leave out how to exactly to that, as there are simply to many options available. Your favourite search engine will help you out.

Unfortunately, there is no way like in Angular, where the opt out can simply be checked into the repository with `"analytics": false` as part of the `angular.json`. It can only be done via the environment variables. [Honi soit qui mal y pense](https://en.wikipedia.org/wiki/Honi_soit_qui_mal_y_pense).

For confidential projects, this means you need to make sure that every new development environment gets correctly configured (f. e. during onboarding). Also do not forget about your build servers or your `Dockerfile`s. Be aware, that multi-stage `Dockerfile`s do not share `ENV` statements over multiple stages, so the `ENV DOTNET_CLI_TELEMETRY_OPTOUT=1` needs to be added to every stage!
