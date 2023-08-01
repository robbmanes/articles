---
title: What a Video Game for Kids can Teach you about Software Engineering and Infrastructure
published: true
description: A game with child-friendly graphics does a better job of explaining IT roles than I ever thought possible.
tags: automation, games, infrastructure, engineering
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ryshbrupy2ch1j3877rb.jpg
date_of_publish: 02-09-2019
---
# What a Video Game for Kids can Teach you about Software Engineering and Infrastructure
This article is published at:
- [dev.to](https://dev.to/robbmanes/what-a-video-game-for-kids-can-teach-you-about-software-engineering-and-infrastructure-29i1)

I'm an avid gamer.  I play almost everything, and most recently I've been getting in to automation games like [Satisfactory](https://store.steampowered.com/app/526870/Satisfactory/) or [Factorio](https://store.steampowered.com/app/427520/Factorio/).  After hundreds of hours in building factories in these games my Steam recommendations decided to show me [Autonauts](https://store.steampowered.com/app/979120/Autonauts/), and at first glance I was a bit appalled at the recommendation.  It seemed very childish and the playground-toys theme turned me off of it for a while, until one day I decided to pick it up and, over the next one hundred hours of play, realized it was the absolute best simulation of *all* of the facets of IT Infrastructure I've ever seen.

Simulations of real-world engineering and software exist everywhere in video games these days.  I was shocked at how extensive you could make faux electronical devices in [Minecraft](https://www.minecraft.net/en-us) when the Redstone update was introduced, and a quick search on your favorite web browser will show people have made entire in-game CPU'S and complex computational devices in massive feats of computer engineering - all within a game about surviving and crafting.

There are quite a few *pure* programming games out there; I love [Zachtronics](https://www.zachtronics.com/) games, which between [Shenzen I/O](https://store.steampowered.com/app/504210/SHENZHEN_IO/), [TIS-100](https://store.steampowered.com/app/370360/TIS100/), and [Infinifactory](https://store.steampowered.com/app/300570/Infinifactory/), can satisfy my need for programming outside of work.  I'm a huge fan of pure logic games as well such as [Baba is You](https://store.steampowered.com/app/736260/Baba_Is_You/), which looks playful but carries a deep approach to puzzle solving utilizing rulesets.  This being said, the target audiences for these games are clearly those who enjoy logic puzzles and already have some experience in them, perhaps an older-than-teenager audience.

Back to Autonauts.  I launched the game with the images I'd seen so far in mind; everything was blocky and cute, and everything had a smile on it in a preschool-wonderland sort of way.  Like any other "Survival Crafting" game, was put in an untamed area and had to scavenge for basic resources to begin the progression from humble vagrant to planetary tycoon as so many other games encourage the player to do.  I made a workbench with my own hands, made an axe, and started chopping down trees and refining logs into planks.

![Welcome to the middle of nowhere, population you.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fgpcfhrok6bz12tfhbmx.jpg)

After the initial work, the tooltips encouraged me to craft a workbench specifically to make robots, or "Bots".  I did so, and then used it to craft my first Bot, and then the true genius of Autonauts began to shine.

Using my new Bot, I was able to "Teach" it to chop trees for me by simply mirroring the action.  As I did so, the Bot translated this into a [Scratch-like](https://scratch.mit.edu/) language on-the-fly with simple instructions that I could click and drag, including the addition of loops or if/else statements with drop-down qualifiers.  It was very simple, and then after a few more Bots and a few more sessions of teaching and rearranging codeblocks, I had a fully automated forestry operation going, complete with re-planting.

![Look out new world, the gentle touch of automation is here.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/btubve7in9uemeqi8ish.jpg)

At this point, I realized I likely never had to chop down a tree by hand in this game again.

Things began to quickly accelerate.  I had quarries of stone and clay worked by machine hands, courier networks delivering to a central warehouse, and teams of assemblers/crafters distributing tools such as axes, picks, and shovels to the various resource sites, each managed autonomously and independently in teams of Bots.  Bots need to be "recharged" by turning a crank, and while I did this by hand, after a hundred Bots were made I made a special team of Bots to turn the cranks on other Bots, removing all of the human intervention everywhere I could in a flurry of automation.

Eventually, after certain milestones I received upgrades; I could now craft machines that made "Colonists" and with that came a lot of new technologies to dabble in such as Agriculture.

I now had "Colonists" to take care of, babies spawned out of a machine that needed food, housing, and clothing.  I set up these teams quickly, creating Bot-tended fields of berries and lettuce, and setting up orchards of Apple trees.  I created a fishing team, catching and slicing up salmon, and fed everything to the colonist-babies I suddenly was responsible for.  My automatic empire grew to support these colonists, and in return colonists produce a resource called "Wuv" which really are just tokens you can put in a sort of Research Machine to advance your technological level, letting you build new machines and better Bots.  Eventually even the *colonists* themselves leveled up, growing into having more and more needs, and I had to adapt previous teams of Bots and make entire new teams to hit the ever-shifting target of needs for these growing colonists.

![Birds eye view of the craziness.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/72vms807ee67xze7bcu5.jpg)

It wasn't until I had to do my first very-real upgrade problem of infrastructure that all of the similarities of my experience in IT, Development, and Infrastructure began to figuratively bash my head in.  The problem was this: I had a large number of entry-level Bots which are slow and have multiple drawbacks, but were otherwise working just fine.  I had intermixed teams of Bots of varying levels and effectiveness, and it was creating intermittent poor performance for one of my teams as sometimes there would be a surplus of materials and other times a drought as the different-level worker Bots performed tasks at different rates.  I sought to consolidate skill levels by upgrading teams to the new `Mk 2` level of Bot, and decommission the old ones, and went about doing so.  There were a few hiccups, but I had felt I had everything under control until I scrolled to a side of my base and saw this:

![Behold, the Salmon abomination.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/emg4wb1wlgn9k64hels2.jpg)

That is a tower of stacked and still wriggling salmon that is reaching the stratosphere.

I couldn't find the problem immediately.  I went to the fishing teams, and couldn't see which unit was responsible for the salmon; I had just replaced my older Bots.  So then I realized I had switched the duty to "fetching" the salmon to my Kitchen team, which led me to the culprit; the sole "Salmon Fetcher" Bot had a "If crate of salmon is full" instead of "If crate of salmon is NOT full" clause which caused it to run once and then ponder what to do ever after.  Therefore, the fisher Bot just kept fishing, and piling the salmon on top of each other, forever.  I thought I had directly copied this code to fetch salmon from an older Bot but clearly I had changed something incorrectly in the course of my upgrade, and really had no way to know what was different.

This isn't the only example I could make of this being a direct allegory to positions in DevOps, Product Manager, Development, QA, Support, or any really software infrastructure position.  The Colonists are really just customers who have the very-real shifting expectations of delivery as their needs are being met in various ways.  Upgrading my infrastructure was always a hassle, and I had to make decisions on when and where to impact my production based on what was most important to me at the time.  I had to decide which Services mattered most, and observe multiple failures to ensure that all edge-cases were caught lest suddenly there be unexpected downtime in production until the issue was caught, rectified, and prevented for the future.

One specific, important example I'd like to share is when I had to shift my "services" from a monolithic model to a microservice model.  As my requirements and physical areas to access expanded, it became better to have resources not all made in my central "factory", so I undertook an effort to move certain types of goods to be made in different locations; tools would be built in the "tool" factory, food and such would be processed in a specific location, etc.

There are so many examples of how this game is the perfect, most real-world example of working in IT that I can't list them all.  It is a children's learn-to-programming game that I found to be an excellent (and intensely fun) description of what working in IT Infrastructure is like, from a myriad of positions and roles which you have to take upon yourself.  The game has no real consequence of failure; your colonists/customers can't leave you or "die" (yet, the game has a new "survival" mode coming out apparently) so it's very forgiving, especially when you forget to feed your customers for a week or so.

All in all, I'm overjoyed at finding this game, and I encourage everyone who's interested in this sort of thing to pick it up.  It seems fairly age appropriate for all levels, and it truly is the best *real* simulation of my experience working in IT so far.  Bravo to the developers.  By the end of my 100 hours, I'd advanced decently far (I am a slow perfectionist in these games) but still have plenty to unlock, but am quite happy with my operation thusfar.  Perhaps this isn't your thing, but I found this to be truly an exceptional example of "what do I do for a living" in a gamified, simplified way.

![Bots making bots.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2pv24881b1urflfayrlb.jpg)

May you catch your bugs early and often, lest they pile up in a tower of wriggling salmon that pierces the heavens.