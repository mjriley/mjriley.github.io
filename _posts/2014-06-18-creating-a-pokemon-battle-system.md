---
layout: post
title: "Creating a Pokemon Battle System"
modified: 2014-06-18 03:49:04 -0700
category: []
tags: [pokemon gamedev unity csharp]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---

## What's this?
I played through Pokemon X and then thought -- I can do that! So I made a replica of the Pokemon X battle system in Unity/C#. My initial intentions were to just create something very simple in a few days, but the project quickly morphed into something meant to mimic the actual game as close as possible (including the graphics).

For the TL;DR -- check out the repository [here](https://github.com/mjriley/BattleSystem), and scroll to the bottom to test out the game.

## Surprisingly Complicated
I finished a workable system in a few days, but I had no idea how complicated the full system would be. There are a lot of mechanics, especially if you're trying to faithfully recreate each pokemon's moves. I suppose my initial impression was that all pokemon just have these basic damage type moves. That's true, but many of those moves have unique corner-cases that make creating a generic framework fairly difficult.
Some of the moves that gave me headaches were:

* Rollout -- the pokemon attacks with the same ability for up to five turns, until he misses. The move's power doubles after each successful hit
* Thrash -- the pokemon attacks for 2 or 3 turns with the same ability. At the end of those turns, he becomes confused.
* Fury-Swipes -- the pokemon attacks 2-5 times, with an unequal probability distribution.

Again, the moves taken alone don't seem that unreasonable. It was simply tricky to create a common interface that would handle all of these edge cases. Ultimately, I decided that each move had the possibility for on-hit effects and on-completion effects, and gave the user the possiblity to create their own delegate methods to provide new functionality. Moves like Rollout required defining a variable power structure as well. I'm really happy with the result, however, due to the flexibility. I think its pretty cool that, even though there are clear standards for what is and is not a move in the Pokemon universe, if I wanted to create a move that repeated for 100 turns, my system could easily accomodate that -- you simply pass in the execution count as a parameter to my RepeatMove class. If you want to create a move that applies three statuses and increases the user's power, no problem -- just create a composite effect for its on-hit ability, with each of the individual effects referenced.

Towards the end of the project, I allowed myself to look at some of the other open-source solutions -- [Pokemon Online](https://github.com/po-devs/pokemon-online) and [Pokemon Showdown](https://github.com/Zarel/Pokemon-Showdown), and was pleased to see that their systems did not share the same flexibilty. For example, Pokemon Showdown implements Rollout as a status -- the Rollout move applies a 'locked in' status to its user, and I believe its power is also stored on the pokemon. This works, but I found it problematic for two reasons. First, it spreads out the logic for the Move. A good chunk of the rollout logic is now stored on the Pokemon rather than inside of the move itself. That coupling increases the difficulty of modifying the two classes, and it also makes the code more difficult to read; anyone reading the Pokemon code is now distracted by the likely superfluous Rollout code, and anyone reading the Rollout code doesn't see the actual implementation. Second, it limits the programmer's ability to create new moves. In this case, because Rollout is relying on the locked-in status to know to continue, it restricts my ability to create a compound ability that does something else -- if I wanted to do something for the first 3 turns, and something else for the first 5 turns, both of those 'things' would require the locked in flag, and so I'd be out of luck.

## 3DS Unity-friendly 'Screens'
Another area I felt went well was the battle prompt screens. Initially, I had one big 'BattleDisplay' class that had the logic for every screen it would show, keying those screens based on the state of the underlying battle system. As I continued to work on the system, this solution felt more and more awkward. Obviously, it wasn't scalable. If my system eventually involved 100 different screens, I *could* store all of those screens in the same class, but it would be impossible to read or maintain. Also, each of the screens was a nightmare to tweak, and the individual screens started polluting the display class with unnecessary variables. The best thing about Unity, to me, is the run-time fiddling you can do to a screen's public variables. I think it helps quickly create user interfaces. However, the solution I was working with was deterring me from doing that, and even when I did tinker, it was a pain to manipulate the display into a state where it would show the desired screen.

I'm pleased with the current screen solution, because I feel it is a good representation of MVC. I ended up creating separate display classes for each of the screens that could be displayed. These screens were descendents of MonoBehaviour, so I could easily load them up and tweak them through the Unity UI. I also made sure not to tie any logic to the screens themselves. Instead, each screen had an underlying controller which was specific to the request made to get there. For practicality, this allowed me to re-use screens when they re-appeared during the scene, even if they were being used for a different purpose (for example, a pokemon's moves are displayed both for selecting which move you'd like to use, and which move should have its PP restored). In such a case, there would be different controller classes, but the screen would remain the same. Obviously, adhering to the MVC pattern also meant that if I needed to change what my game looked like -- I designed it to replicate the 3DS, but maybe I'd like to make it more iOS friendly -- I would be able to re-use all the same controller logic just by providing a different set of screens.

## Moving Forward

### Testing
There is still a ton I'd like to improve with this project. I really wanted to incorporate testing in this project, as in the past I've heard that Unity is anti-test by nature. I learned that this wasn't true; if you focus on separating your logic from your UI, the logic becomes much easier to test. That said, I think testing works best when you are writing your tests as confirmation of your requirements. When you are you not sure what your requirements **are**, writing tests begins to feel counterproductive. The scope and intent of my project changed as I was going along; in the future, I know I will need to plan much more carefully to faciliate better testing.

### Pure Logic
Again, the intent changed during development. I think my initial idea was just a simple recreation of the 3DS game. In that environment, it was strictly your human-controller player vs an AI opponent. Because of this, there are assumptions within the code about who can respond when, and who gets what output. In the future, I think the logic should be made to be purely agnostic of the type of player playing. This allows the library to be used for an online head-to-head simulator as well as for Human-vs-AI gameplay. I created some classes to support this agnostic behavior; for example, requesting a player's next move is performed through a generic interface. Hopefully, it is not too much work to fully realize this. The end-goal would be two separate projects -- the logic library as a C# DLL, and the game logic and GUI as the unity project.

### Always Room for Improvement
Code never seems truly finished -- there is always something to clean up or refactor. Certainly, that's the case here. As I worked through the proper 'Unity' way to do things, I think the structure of my code flexed quite a bit. I know there are many places in the code right now that could use some love -- some code needs to be rephrased, some of the earlier screens should use different logic, etc. There's always tomorrow...

<script type="text/javascript" src="https://ssl-webplayer.unity3d.com/download_webplayer-3.x/3.0/uo/jquery.min.js"></script>
<script type="text/javascript" src="http://webplayer.unity3d.com/download_webplayer-3.x/3.0/uo/UnityObject2.js"></script>
<script type="text/javascript">
	<!-- 
	var config = {
		width: 400,
		height: 500,
		params: { enableDebugging: "0" }
	};

	var u = new UnityObject2(config);

	jQuery(function() {
		var $missingScreen = jQuery("#unityPlayer").find(".missing");
		var $brokenScreen = jQuery("#unityPlayer").find(".broken");
		
		$missingScreen.hide();
		$brokenScreen.hide();
		
		u.observeProgress(function(progress) {
			switch (progress.pluginStatus)
			{
				case "broken":
					$brokenScreen.find("a").click(function(e) {
						e.stopPropagation();
						e.preventDefault();
						u.installPlugin();
						return false;
					});
					$brokenScreen.show();
					break;
				case "missing":
					$missingScreen.find("a").click(function(e) {
						e.stopPropagation();
						e.preventDefault();
						u.installPlugin();
						return false;
					});
					missingScreen.show();
					break;
				case "installed":
					$missingScreen.remove();
					break;
				case "first":
					break;
			}
		});
		u.initPlugin(jQuery("#unityPlayer")[0], "/assets/battleSystem.unity3d");
	});


	-->
</script>

<style type="text/css">
<!--
div#unityPlayer { cursor: default, height: 500px, width: 400px }
-->
</style>

<div class="content">
	<div id="unityPlayer">
		<div class="missing">
			<a href="http://unity3d.com/webplayer/" title="Unity Web Player. Install now!">
				<img alt="Unity Web Player. Install now!" src="http://webplayer.unity3d.com/installation/getunity.png" width="193" height="63"/>
			</a>
		</div>
		<div class="broken">
			<a href="http://unity3d.com/webplayer/" title="Unity Web Player. Install now! Restart your browser after">
				<img alt="Unity Web Player. Install now! Restart your browser after" src="http://webplayer.unity3d.com/installation/getunityrestart.png" width="193" height="63"/>
			</a>
		</div>
	</div>
</div>
