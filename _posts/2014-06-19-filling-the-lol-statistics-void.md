---
layout: post
title: " Filling the LOL Statistics Void"
modified: 2014-06-19 05:10:13 -0700
category: [Programming]
tags: [LOL, Statistics]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---
Alright, here's a new project proposal -- A website that allows you to see see how your item, rune, and mastery choices affect your damage in various ways. League's a great game, but, given that a lot of the community is coming from the WoW world, where we were obsessed with DPS meters, its a bit strange that damage statistics have seen so little attention.

## The Goal
I want to give players a way to reason about how certain champions stack up against others, within the same role. I want someone to be able to go to my site, and chart Corki's damage against Caitlyn's, in a way that allows the person to realize -- "Oh hey, from Level 1-6, I'm really strong. From 6-10, we're on the same level, and then beyond that, I can't compete." Maybe that realization is obvious to most marksmen mains, but how about this one -- "Hmm, I never have problems as Caitlyn in my early game matchup against Corki, but I'm not thrilled with my late game damage output; I wonder if switching to armor penetration reds would allow me to get thru the early game, and I wonder how my damage would then stack up in the late game against other champions?". Specifically for marksmen, we're just very ignorant. In general, we tend to create this global marksmen page which gets used for every AD we play, whether its optimal or not. And...embarassingly, sometimes we're just completely ignorant of power trends. For example, Ezreal, Lucian, and Twitch all had periods where they slipped under the radar before we realized they were dominant champions. Their dominance isn't magic. Sure, in the case of Ezreal, some of his dominance will always be in his evasion kit, which is very difficult to quantify. But for Lucian and Twitch, there is certainly data that we can visualize to understand why the champions are as popular as they are -- and if there are champions we might be missing out on that can function in a similar fashion.

## The Vision
The finished product revolves around visualizing this data. The most obvious thing to me is to have it displayed in a line graph. Create a line graph, with the X-axis reflecting a champion's level, and the Y-axis reflecting the desired statistic. Initially, for something like base attack damage, the graph will be linear, as the only change per level is the attack damage per level the champion gains innately (plus the potential gains from per-level runes and masteries). However, the user should be able to double-click on this line, at any point, to create 'inflection points', similar to how you might modify a bezier curve in Photoshop. These inflection points allow the user to customize the champion's statistics. Inflection point data will carry its effect through to every subsequent data point/champion level. For example, you might create an initial inflection point at Level 1, which will modify your intentory with a Doran's Blade. The entire graph should now rise, as each level's damage is affected by the extra damage it provides. You might create another inflection point at 4, for your first back, where you pick up Vampiric Sceptre, and then at 6 or 8 to reperesent a BF sword. All of these should show a noticeable spike in your damage on the graph going forward.

That's fine for a single champion, but the goal, again, is to compare champions -- to understand why one is more appropriate for a given situation than another champion. To that end, it needs to be easy to compare two (or more) champions in the same situation. In the previous example, there needs to be a "Create new line" button, which will create a second line that reflects the same data as the first. However, they are true copies. With this 2nd line, I can change the champion to a different one; all my item choices will remain at their inflection points, but now the entire line will adjust to reflect the base values for that champion.

It all needs to be customizable. I need to be able to add, modify, and remove the existing inflection points. This is important if I were trying to compare Caitlyn against Vayne, for example, since they have 2 very different build paths. I'd carry over the item choices I made on Caitlyn to Vayne's line, but then I'd probably need to adjust the BF sword purchase point, for example, to reflect that Vayne is trying to rush BotRK rather than an IE/BT. It would be nice to also be able to freely move those points around -- drag one inflection points to the left or right, to change the level I expected the point to happen.

This wouldn't be limited to just champions, however. Item choices are equally important. Especially now, with the newly adjusted marksman items, most of us don't really understand the tradeoffs of our itemization choices. Say I play Lucian -- where am I stronger, and where am I weaker, over time, if I choose to rush BT/IE or BotRK? This time, the graph will display 2 lines, but both will be for the same champion. Our modifications will only be for the build paths, but the lines will still reflect the damage difference that we want to see.

## Considerations

### Scope
Ideally, I want this project to be finished pretty quickly -- say 1-2 weeks. There are a lot of different statistics I could analyze, and if I did all of them, the scope could quickly spiral past this window. Accurate combo damage, for example, is something I've wanted to do in the past, but is just super time-consuming -- you need to test to figure out the delays between auto attacks and spells for every champion. Because the whole graph idea remains the same regardless of the stat, I think I can start out simple, with something like raw attack damage and effective health, and then add on as I go. In the end, the statistics likely will have to reflect both the attacker and the defender, but I can ignore the defender to start with.

### Blogging
I'm treating this project differently. With my previous Pokemon project, I never expected to talk about it until the very end. But, even making notes to myself of aspects that were worth talking about, when I finally reached the end, I found summarizing the entire project difficult, because:

* Sheer length -- if I documented all the 'interesting' problems I ran into, who would actually read it? The bulk of the post would make it difficult to read, and deter people from even trying.

* Fatigue -- just finishing a project like that was tiring. Having the burden of then remembering all my issues and summarizing them wasn't something I actually wanted to do.

* Memory issues - even with notes, the most recent problems I ran into always seemed the most relevant to talk about, while my earlier struggles just seemed unimportant. This was true even if I was really fascinated with the problem previously.

To that end, I'd like to blog frequently about this project, as I make progress. The entire length will still likely be long, but you can pick and choose what to read. Small, frequent blog posts shouldn't burn me out, and I'll have a better idea of what I'm describing because it will be current.

### Testing
I swear to God, **THIS** time, I'm going to actually succeed at doing real TDD. I think it's always a little unclear how to do good testing when half your project is GUI behavior, but if I keep my logic separate from my view, I think I'll be in good shape. I've now made the concession to myself that when I put off my tests until after I write the code, the process seems very time consuming, so I think I can use that as motivation to force myself to write the tests first. I'm also intrigued on how it will affect the code I write.


## Requirements
So, let's get started with my first verison's requirements

* I must be able to plot champion data on a graph.
* The graph must be divided by champion level on the X-axis, and the statistic on the Y-axis.
* I must be able to view a champion's damage as his level increases (we'll start there).
* I must be able to create inflection points on that graph for new item purchases
* I must be able to delete inflection points
* I must be able to modify inflection points -- modification means that all item choices can be modified, including removing existing items.
* I must be able to shift my inflection points left or right to different levels.
* The graph must reflection the effect of those inflection points on the statistic.
* I must be able to copy my existing data into its own line, so that I can modify that data independently from the previous one.
* Changes to one line must not affect any other lines.
* Lines must be displayed in different colors.
* I must be able to remove existing lines.
* I must be able to change the champion for a given line.
* I must be able to change the runes for a given line.
* A line's data must update to reflect its inflection points, even when those points change or get moved around.
* All data must be accurate -- item statistics, champion statistic values, masteries, runes, etc, must all have their values as they appear in the actual game.
* At a minimum, a small subset of marksmen must be supported (Caitlyn, Ashe, Tristana to start with due to their AA focus).
* At a minimum, a small subet of marksmen items need to be supported.
* At a minimum, attack damage, attack speed, and armor penetration marks and quints must be supported.
