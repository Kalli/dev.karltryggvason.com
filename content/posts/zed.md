+++
date = '2025-10-23T09:11:17+02:00'
title = 'Zed'
+++

For the last couple of months I've been trying out the [Zed](https://zed.dev) editor. So far I'm quite happy with it. It's open source, really fast, flexible, has good ai support, a helpful community and a growing ecosystem of extensions.

I'd been a VS Code user for the last couple of years but it often felt sluggish to me. So the initial selling point of Zed, was its speed (and it is _very_ fast compared to VSCode). Using the VS Code keymaps meant that I was able to transfer most of my muscle memory shortcuts over without any pain. In the beginning there were a few VS Code features and extensions that kept me from switching over 100%, but since then the Zed team have added support for what I missed the most (git support and the debugger were two big ones) and seem to be moving pretty fast in terms of both features and ecosystem growth.

I think the original Zed pitch was the speed as well as [collaboration](https://zed.dev/docs/collaboration) and "multiplayer" workflows which look pretty cool (I haven't really tried them yet as I don't work with a lot of other Zed users). But lately, like so many others, they have shipped a lot of ai related features. They aren't yet another VS Code fork though and I find that I like and agree with their their way of adding ai support (keeping it optional, open, granular and configurable) and their way of thinking in general.

I'm still not fully onboard with the agentic ai workflows, I still find them a bit scary and haven't found a good way to isolate or sandbox them in a way that feels robust but easy to manage. So I limit [tool access](https://zed.dev/docs/ai/tools) to reading and searching for files (disallowing the editing of files and executing code, mostly using the "ask" profile). But having the agent panel there and being able to directly reference files and context was a big jump for me from going back and forth to a chat tab in the browser. Being able to bring my own api keys was a big factor too, I don't feel locked into any one provider or model.

I also like their [tasks](https://zed.dev/docs/task) functionality that neatly lets you combine the editor interface to command line tools. When I'm working, I often set that up to execute specific tests, run some [sql clean up](https://dev.karltryggvason.com/sqlyac-structure-and-tooling-for-your-sql-files/) or http requests, related to whatever feature or bug I'm fixing. That way I can set up complex scripts but (re)run them repeatedly with ease without leaving my editor.

Overall I'm quite happy with Zed. I think I will be sticking with it for the foreseeable future. If you are in the market for an editor, I recommend trying it out.
