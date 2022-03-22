---
title: Why we're ditching Bukkit
date: '2022-03-18'
tags: ['dyescape']
draft: false
summary: An introduction into our journey of removing Bukkit entirely
---

Bukkit, dubbed "Bruhkkit" within the team, is a Minecraft server modding API that can be found on practically every
Java Minecraft server. Note that for this blog, we'll also speak of any extension, continuation, or fork of Bukkit such
as Spigot and Paper. In this blog, I'd like give an introduction into why we're making significant investments to remove
Bukkit completely, and what some of the concerns are that we currently run into.

## The good

Bukkit is great at exposing reliably API layer to expose an otherwise obfuscated Minecraft codebase, and allows creators
to modify their server to create different games using Minecraft as a sandbox. There are tens of thousands of Minecraft
servers, some staying closer to the vanilla gameplay than others. Dyescape is no exception, as we've built an MMORPG
within vanilla Minecraft using Bukkit.

## The bad

While Bukkit is great at being a modding API, it remains exactly that; a _modding_ API. Modding implies that you're
taking an existing base, and you modify it to your liking. Unlike a framework, Bukkit doesn't provide you with a toolbox
to build something new from scratch, but rather exposes a layer to modify what already exists. What exists, is Minecraft
as a base game; with all the vanilla, mostly survival gamemode oriented features. For the majority of Minecraft servers,
this is fine. Survival, factions, prison, skyblock, or even some larger minigame gamemodes are fine with using Bukkit
as their base, as they expand upon the base game rather than throwing most of the base game features out the window.

Dyescape, however, is the opposite. Breaking, crafting & placing blocks, exploring randomly generated terrains & cave
systems, killing randomly spawned mobs with static loot tables, surviving the first night in a dirt house until your
newly designed mansion is ready, growing crops & milking cows; none of those vanilla features exist in Dyescape.
Instead, you play through a carefully crafted custom world, featuring a story with actual characters & quests.
Throughout your journey, you may find an arsenal of custom items, unlock powerful skills, and get accompanied by an NPC
or a group of friends to raid a dungeon.

Most famously, our phasing solution causes the biggest concerns. Phasing allows us to create server-sided isolations,
meaning two players can be in the exact same location and see different entities & blocks. This gives us an incredible
opportunity of implementing more complicated content which wouldn't have been possible otherwise. Minecraft at it's core
has no support for this, and is not developed in a way to easily support this. Although we managed to build it on Bukkit,
it involves a lengthy & difficult process of injecting these isolations checks in a large number of places in the vanilla
codebase. It's a contributing factor for sticking with Minecraft 1.16.5 until beta, as updating it to newer versions
causes phasing to break, and needs to carefully be updated & tested intensively.

As a result, while developing the Dyescape, we spend significant effort disabling, adjusting, removing or working around
existing vanilla features. Rarely do we implement a feature where we are not impacted by the vanilla way of work. This
creates unexpected bugs, limitations, performance issues or even potential exploits that we have to account for, and
eventually translates into more effort being necessary to develop & maintain our services.

## The ugly

While the development of the game is not as productive as we would've liked it to be, there are bigger concerns.
Minecraft is not a lightweight game in how it was implemented; despite coming out in 2010, some players still struggle
with running the game in a performant matter, and it isn't even the Night Watch in terms of graphics. Unfortunately,
the back-ends suffer even more. As a Bukkit server handles more players, it gets increasingly higher load. More players
join, they get spread around the world, causing more chunks to be loaded. With all of the vanilla features still baked
into the system, loading more & more chunks makes the server perform more & more calculations that we don't need.
While some of these features can be optimised or even disabled entirely, it doesn't fix core of the system being too
heavy.

An up-to-date Minecraft server, and inherently Bukkit, is heavy on almost every hardware aspect; it needs bulky single-threaded
performing CPUs (such as a Ryzen 7 5800x or similar), 12 to 16 _gigabytes_ of memory, at least an SSD (preferably NVMe)
and a 200 - 300 mbit connection just to run 200 - 300 players. Naturally, these specifications differ per gamemode, but
as Minecraft gets bigger & better every major release, hardware requirements increase. For example during the Caves &
Cliffs update, chunks have been expanded vertically, allowing caves to go deeper and mountains to peak higher. Requiring
such an excessive amount of hardware to serve a couple hundred players is not feasible, especially considering the high
resource usage exists because of features we don't need.

If you thought less productive development & increased operational cost was bad, unfortunately, it gets even worse.
We wish to eventually have hundreds, if not thousands of concurrent players. Doing so requires infrastructure and
operational management to keep the game as accessible and stable as possible. From time to time, we may need to install
updates, or downscale instances when less players are online. This isn't so much a concern for minigame servers for
example, where most instances have a very short lifecycle. You play a minigame for 10 minutes, sit in a lobby for two
minutes, then join another 10 minute minigame. Such servers can perform rolling updates in the background where
instances are recycled or removed once everyone has disconnected from said instance. In an MMORPG, the lifecycle of an
instance is far greater, while also having far higher playercounts as opposed to a small minigame instance. Players may
enjoy the game for hours on end while remaining on the same instance. Bukkit, and surrounding proxy tools such as
BungeeCord, don't support one instance seemingly taking over the work from another, swapping players over to a new
instance without the player noticing a thing. This makes good operational management at large scale impossible. It
would translate to downtime every time we wish to install an update. This downtime should be communicated to players,
and maintenance should be executed on times when we have the least amount of players; likely deep into the night. It's
not something we want to bother the players or the team with.

## The future

Almost immediately after our first alpha launch (v0.1.0), we opened up discussions within the team for a new project
to tackle the above, and to ready ourselves for a future with better server software specifically developed for our
use-cases and scenarios. This project has been dubbed [Kubus](https://i.imgur.com/eQcRytC.png), and is scheduled to
be used as part of beta and onwards.

Kubus will be a distributed game server software written in Kotlin, integrated into Kubernetes & Agones, supporting
seamless relocation of players through the use of fallback & replication instances, with protocol termination to
loosely couple ourselves of Java Minecraft, should we ever wish to support a different client. Storage & other internal
systems will also be rewritten to support Command-Query-Responsibility-Segregation (CQRS) and Event Sourcing, for added
stability, scalability & analytics.

If you've got some spare time, and want to help out with what might just become the most technologically advanced Java
Minecraft server; we could use the hands.
