---
title: 'Looks Good To Me (Says Claude)'
date: '2026-03-07'
draft: true
---

The past week, I have been tampering a lot with Claude Code, well, at least till I ran out of tokens. Now I'm back to meat-produced code (that's when you use the ~1400g heavy slug in our head, which is accidentally the most complex object in the known universe) —* till my weekly limit resets.

_* Yes, this is a human-produced em dash, simply because I love typography!_

But we are no [promptsitutes](https://www.urbandictionary.com/define.php?term=promptsitute) anymore; we are doing spec-driven now! If this sounds familiar to you, then you are not too far off! There are a lot of similarities with UML-driven development from the 90s —* and a lot of waterfall!

_* Another human-produced em dash, and this time I didn't even need to search for it on the internet!_

## No agile no more?

Spec-driven works by writing the specification upfront, or to phrase it better: you instruct an LLM to write the specification for you and then tell another LLM to implement it, which, of course, you let another LLM review:

```
┌───────────────────────┐     ┌─────────────────────┐     ┌───────────┐     ┌─────────────┐
│                       │     │                     │     │           │     │             │
│ Requirements Engineer ├────►│ Technical Architect ├────►│ Developer ├────►│ QA Engineer │
│                       │     │                     │     │           │     │             │
└───────────────────────┘     └─────────────────────┘     └───────────┘     └─────────────┘
```

So now you can do the process-heavy inefficiencies of enterprises as a single developer! Isn't that awesome?

## Intelligence disclaimer

Before we go into the experiment I did, let's set some things straight: artificial intelligence is, in fact, not intelligence. It is statistics on steroids. That's not something inherently bad, and there are a lot of use cases for it; just please stop assuming it understands _things_.
Also, it has no consciousness, even when the “Trust Me Bro” tech CEOs like from Anthropic are gooning about it at night:

!["Breaking: Anthropic CEO says Claude may or may not have gained consciousness, as the model has begun showing symptoms of anxiety."](/assets/posts/anthropic-ceo-lost-consciousness.png)

Please stop assigning living, breathing, human traits to things, which they are not.

It just predicts the most likely next token, which, again, can be good enough, especially as most software seems to run on that principle anyway nowadays, but it can also go horribly wrong:

!["Claude Code wiped our production database with a Terraform command. It took down the DataTalksClub course platform and 2.5 years of submissions: homework, projects, and leaderboards. Automated snapshots were gone too. In the new newsletter, I wrote the full timeline + what I changed so this doesn't happen again. If you use Terraform (or let agents touch infra), this is a good story for you to read."](/assets/posts/claude-code-wiped-production.png)

_(Coincidentally enough, I collected both of these two screenshots within hours today.)_

## The setup

There have been a lot of vibe-coded demos by creating new greenfield applications, which, in my opinion, is already the first problem, because the world runs on brownfield software.

Therefore, I took an application of my own and let Claude Code steer the wheel. The application in question is of simple to medium complexity that I use for invoicing, time tracking, etc., which I wrote when I founded my company (because why would you buy software when you can write it yourself, right? RIGHT?).

Anyhow, it's about seven years old, and while it got some maintenance in the past years, it could need some love. The tech stack consists of **.NET, ASP.NET Core, EF Core, Angular etc**. So time for some technical upgrades! (a use case that probably a lot of companies out there have)

For this experiment, I resorted to using **[OpenSpec](https://github.com/Fission-AI/OpenSpec)** as a lightweight spec framework. I would also recommend setting up **[serena](https://github.com/oraios/serena)**, which is an MCP server that enhances your LLM with tools for querying code (just like in an IDE) by utilizing language servers to save tokens.

I'll skip detailed instructions here, as this post is about results and not how-to, but I recommend checking out the [live streams](https://www.youtube.com/playlist?list=PLw0jj21rhfkNPdETKsoyPEbKkZpCpB4Gh) [Dylan Beattie](https://dylanbeattie.net/) and [Emmz Rendle](https://rendle.dev/) have been doing.

## Slop in, slop out?

First, I started by generating a `CLAUDE.md`, so the LLM has a summarized view of the project structure for further work. Then I created multiple personas (system prompts) for the roles as described above. Yes, this was all generated. I went full sloperator to test the limits of these tools.

In phase 1, I did brainstorming with the _requirements engineer persona_ to gather the requirements about what needs to be done. This was quite an iterative process, and while the LLM brought up a lot of good questions, I only later realized that a couple of things were missing or actively marked as further proposals without asking if those things should be included.

Phase 2 with the _technical architect_ was quite similar, just refining the requirements and defining technical boundaries, but again, important migration steps were missed, which I only realized later. It also made a wrong assumption, which it only caught during implementation.

More or less, it came down to the following changes:
- `Microsoft.AspNetCore.SpaServices.Extensions` → `Microsoft.AspNetCore.SpaProxy`
- `.NET 8` + `Startup` → `.NET 10` + `Minimal Hosting`
- `Microsoft.AspNetCore.ApiAuthorization.IdentityServer` (which includes an older version of `Duende.IdentityServer`) → `OpenIddict`
- `Angular 18` → `Angular 21`
- `Angular Standalone Components`
- `Angular Signals & Zoneless`

Now that I had specifications that _looked_ good, it was time for implementation!

## The human in the loop

Phase 3 and phase 4 were the implementation and review steps, which happened independently for all changes listed above. Again, I went full sloperator, trying with minimal interference to fully test the limits of these tools.

In all cases, the implementation was fully done by Claude. The review usually consisted of some manual smoke tests by me and an automatic review by Claude.
In my opinion it is also quite funny that the LLM that wrote the code finds issues in its own code (just by using a different system prompt).

### `Microsoft.AspNetCore.SpaServices.Extensions` → `Microsoft.AspNetCore.SpaProxy`

This one was the start, and it horribly failed in multiple ways. In all fairness, in the end Claude was able to fix it, but it took many, Many, MANY iterations, burning through tokens. It took 2-3 “sessions” (read: till Anthropic decides you used enough tokens) to actually solve this one, and I had to crank up the model Opus 4.6 with intensive thinking, burning even more tokens.

The problem was that the `SpaProxy` simply works differently out-of-the-box by not proxying but actually forwarding to the Angular application. So different URLs (because different ports), the backend was on `HTTPS`, frontend only on `HTTP`, because there was no setup for `HTTPS`, resulting in all kinds of CORS errors and configuration issues. And as LLMs are good at creating what is likely  correct, that is not something that works very well with configuration that simply **needs** to be correct.

None of this was accounted for by the _technical architect_, and I assume that's the primary reason why the implementation failed so hard.

### `.NET 8` + `Startup` → `.NET 10` + `Minimal Hosting`

After the initial struggles, this one kept me motivated to continue the experiment. It worked mostly flawlessly; I would probably have left a couple of comments in the code review, though. It also ignored a new warning instead of fixing it.

### `Microsoft.AspNetCore.ApiAuthorization.IdentityServer` / `Duende.IdentityServer` → `OpenIddict`

This one scared me the most, simply because it is complex, and not gonna lie, this would have taken me a long time to do. That’s why I was all the more surprised that Claude executed this change mostly flawlessly. It took only two iterations of “Hey, this does not work; here is the error message” to actually make this one work.

I’m confident that this would have taken me more time to do this migration than what the LLM spent.

The main downside of this one is security-related: I have no idea if the changes that were made are actually still secure. They certainly look secure, but as I had almost zero involvement, I could not verify that. The upside here is that I’m the only user and the application only runs on demand (and only locally), so it is no disaster. Theoretically, this application could even do without a login.

### `Angular 18` → `Angular 21`

There were a couple of issues again. Details were missed again during the specification creation because while I mentioned during the discovery phase that I wanted to migrate the test runner to `vitest` (new default in Angular 21), I did not specify that I also wanted to migrate to the new Angular build system (which I implicitly assumed should be part of an `Angular 18` → `Angular 21` 21 migration). In comparison to other things that were explicitly mentioned as non-goals, this one didn’t seem to be a concern at all.

Furthermore, it broke two features, both resulting from package upgrades during the change.

Bootstrap seems to had breaking changes in minor versions again (`5.0.2` → `5.3.8`), and therefore broke the highlighting feature for late invoices, as CSS classes were wrong now.

Furthermore, Claude botched the upgrade of the markdown editor I was using. I was asked if a different library would be okay, to which I obliged, but in the end, the editor was just left commented out, resulting in no editor at all.

As usual, there were some quality issues, leftover files and things that were not used any more.

### `Angular Standalone Components` & `Angular Signals & Zoneless`

I’m going to keep it short: only a couple of small issues, mostly flawless. Just the usual: some code quality issues.

### All the things that were missed

As initially mentioned, a lot of things were missed during the requirements gathering. I would have loved if the following questions had turned up, that I only realized later were missing:

- Linting (“autonomously” decided that it was a no goal)
- New Angular builder (see above)
- New C# features (`ImplicitUsings`, file-scoped namespaces etc.)
- Better package manager
- General more clean-up (just a lot of leftovers that were not used at all/anymore)

## Bonus: a customer application

After these tests, I decided it was time to give it a chance to work on a real project: basically a small additional API endpoint + a new button in the frontend to call it. I did not expect this one to fail, because it was a really easy change, and functionality-wise, it was actually fine, but what killed it was an XML resource file.

The thing is, the application language is German, but all strings are in `XML` resource files so that we could add translations at any point. And as it contains German strings, and me being a big advocate of proper typography, the file contained a lot of strings using German quotation marks („“) instead of the “normal” double quotation marks ("").

Claude apparently picked up that trait and used it for attributes in the `XML` file. It burned through tokens trying to fix it until I interfered and fixed it manually.

Maybe we should add support for German quotation marks and Guillemets in `XML`?

## The agentic way of life?

Okay, time for the conclusion!

I have to agree, LLMs got a lot better at doing coding work since the last time I took a serious look at them. But on the other side, they are still far from being perfect, very far from AGI and also quite a bit away from being real autonomous. In my tests, it still required a lot of human-in-the-loop, but that part shifted heavily from the implementation part to the specification part. Again, they can't “think”, but they work well with very, very detailed instructions, so this makes perfect sense.

All in all, I, however, cannot support claims like 5–7 hours of “autonomous capacity”, as I have seen these days online. The whole workflow still required a lot of intervention in my experiments. Subagents may be part of the answer here, though.

Furthermore, the overall code quality and just dead stuff left behind are still a big problem. Some have already called for that _code_ quality will not matter that much anymore with more autonomous development workflows. It remains to be seen if this will actually be the case. So far, I have been cleaning up after Claude like after an intern. At the moment it is a lot of _LGTM_ (Looks good to me) energy — as reviewed by Claude itself.

_Author's note: no LLMs have been harmed in the creation of this blog post._

## Links

- [Let's Learn Claude Code by Dylan Beattie & Emmz Rendle](https://www.youtube.com/playlist?list=PLw0jj21rhfkNPdETKsoyPEbKkZpCpB4Gh)
- [OpenSpec](https://github.com/Fission-AI/OpenSpec)
- [serena](https://github.com/oraios/serena)
