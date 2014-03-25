---
title: Designing Random Enemy Encounters
layout: post
---
In investigating gameplay systems, I wanted to start small, and I figured random enemy encounters were pretty basic. I experienced them in Pokemon X, and they're in most of the RPGs I played as a child. However, the more I thought about them, I realized there were a few challenges that would affect what seemed to be a simplistic design.

First, let's establish the basic idea of what I want here -- a player should be able to move around an area, and, randomly, encounter an enemy 'native' to that area. I think it is important to note, however, that the behaviour really doesn't need to be completely random, and sometimes it is better that it isn't. If we define the random property here to be the number of steps taken in order to find an enemy, then truly random behaviour would risk a player taking thousands of steps in between encounters, or having encounters spawn back-to-back-to-back with one step in between. Either situation would be fairly frustrating for a player, so the randomness will probably need to have some boundaries.

The first thing is to cover how our area is defined. When I say 'area', I'm thinking of a series of tiles. I can think of two basic solutions:

* Contiguous areas are defined as one entity.
* Contiguous areas are split into individual tile definitions.

There are trade-offs for both solutions:

* If we choose to go with a single entity per area, we save on storage space/memory, but at the cost of algorithm execution time. Whenever I want to check which area a player is in, I need to do some kind of search. Compare this to defining individual tile definitions -- we can represent the entire area as a 2-dimensional array (assuming 2D space), so lookup is constant -- we simply access the array element indicated by the player's coordinates.

* Setup and Maintenance should also be a consideration. Areas are easier to maintain, but clunky to define for 'complex' geometry. For example, a rectangular area with several 'holes' in it would either need to be defined as multiple areas, or we would have to augment the notion of an 'area' -- i.e. now the area would include exclusions, or the area would be defined with a list of connecting points; either case would increase the complexity of a lookup function. If the area was sufficiently 'hole-y', you'd essentially be using a tile solution anyway. In comparison, the tile solution is easy to define, but tedious and hard to maintain. With a full tile solution, you'd likely run the risk of 'missing a spot' when you changed tile definitions. In either case, an editor is ideal for this kind of thing, and it would greatly help the game designers if they could modify the terrain and definitions themselves with some kind of GUI system.

* Finally, it's not generally wise to over-engineer a system, but flexibility might also be worthwhile to consider. While areas are suitable for defining a block of tiles that all share the same attributes, in terms of random enemy encounters; there may still be per-tile information that would necessitate another tile system anyway. For example, in Pokemon, while it seems that the list of random enemies is shared for most of the terrain in an area, there are also per-tile issues like hidden items and poke-radar information. A tile system also just offers more flexibility if you want to create a very specific feel -- perhaps you want to change the frequency of encounters based on the tile, or commonly offer specific pokemon per tile.

Ultimately, I chose to go with a tile solution. I think the main thing areas would have going for them would be the ease of maintenance and the lesser memory overhead, but I'd utilize an editor to minimize the problems of the former, and I'm not even sure I can bank on the benefits of the latter; this is once again a cause for profiling if/when it appears there is a performance bottleneck. I think tiles just offer too much flexibility for the type of systems I've enjoyed in the past (I loved that FF3/6 had very small areas that would define which enemies you could run into).


The next issue to deal with is the random enemy encounters themselves. A basic algorithm would be something like the following:

```
const int ENCOUNTER_THRESHOLD = 75;
const int MAX = 100;

bool isBattle()
{
	int i = rand(0, MAX);
	return (i > ENCOUNTER_THRESHOLD);
}

void handleBattle()
{
	if (isBattle())
	{
		doBattle();
	}
}
```

However, this still suffers from the problem listed above -- the player could be slammed with repeated encounters or not experience an encounter for thousands of steps. Here's a basic modification to fix that problem:

```
int stepsSinceLastBattle = 0;
const int STEP_INCREMENT = 1;

bool isBattle(uint stepsSinceLastBattle = 0)
{
	int i = random(0, MAX) + stepsSinceLastBattle;
	return (i > ENCOUNTER_THRESHOLD);
}

void checkForBattle()
{
	stepsSinceLastBattle += STEP_INCREMENT;
	
	if (isBattle(stepsSinceLastBattle))
	{
		doBattle();
		stepsSinceLastBattle = 0;
	}
}
```

Unfortunately, I don't think either of these solutions is ideal. I see two main problems with this solution:

* Testing/Diagnostics. Generally in testing, we want to eliminate all randomness. For that purpose, we could inject a random generator as another parameter to isBattle, but first...it's really ugly, and second, it won't fix the problem for monitoring. Say we release this game out to the public, and notice that a player seems to be battling very frequently. Unfortunately, due to randomness, there's no good way to confirm this; the random generator might simply be rolling battles more often than we think it should. What we'd ultimately prefer is a system that allows us to monitor the 'randomness'.

* The step increment is only applicable for someone who stays in the same area. If the player leaves that area, the increment should likely be reset. It doesn't *have* to be reset, but when I stop walking in a hostile area, I expect that I'm *safe*. Walking into another area and being immediately bombarded by a random fight, due to my walking in the previous area, seems wrong to me. And, if we do decide to reset the steps, what happens for a player like myself who often likes to hop in and out of an area, walking on the outskirts? Do we once again run into the isssue where I might not have a random encounter for thousands of steps?

Because of these and some customizations, I ended up choosing a different solution. Here's the code I settled on for my unity solution:

```
	void UpdateHostility()
	{
		int curX = (int)transform.position.x;
		int curY = (int)transform.position.y;

		int hostilityIncrement = grid.getHostilityData()[curX, curY];
		int hostilityThreshhold = grid.getHostilityThreshhold()[curX, curY];
		List<PokemonData> pokemonList = grid.getPokemonData()[curX, curY];

		currentHostility += hostilityIncrement;

		currentHostility = Mathf.Max(currentHostility, minHostility);

		// check for a random battle
		if (currentHostility >= hostilityThreshhold)
        {
			TriggerBattle(pokemonList);
			battleOccurred = true;
		}
        else
        {
            battleOccurred = false;
        }
	}
```

This is a completely tile-based solution. 'Hostility' is the term I use to determine whether a player will run into an enemy. The basic concept stays the same -- the player holds his own hostility value, and when it passes the hostility threshold, a random battle is triggered. However, only two random values are ever generated between encounters, which enables more efficient monitoring. Each tile contains an increment -- as the player walks between tiles, that increment is added to his hostility value. If that value surpasses that tile's threshold, a battle is triggered. This allows extreme flexibility for each tile. In addition, it can solve the problem of leaving an area if you design the tiles properly. I have 0 hostility increments directly outside hostile areas, meaning that weaving won't affect your hostility for the nearby area. However, if you continue moving away from the area, more distant tiles have -1 increments, so that you shouldn't run into instant battles upon entering a new area. After each battle, I generate a new minimum hostility and current hostility. The minimum hostility protects against 'predictable' randomness. Since the tile hostility thresholds never change, if I allowed the player to reset his hostility to 0 between battles (by walking in those -1 increment tiles), it would always take x steps to trigger a battle upon entering a new area. The random current hostility simulates the same randomness of earlier solutions with only a single random number. Finally, as I maintain the player's hostility value instead of generating a random one for every step, it allows me to more easily handle items that affect generated hostility (for example, the moogle charm from FF3).

Anyhow, the final implemented concept:

<script type="text/javascript" src="https://ssl-webplayer.unity3d.com/download_webplayer-3.x/3.0/uo/jquery.min.js"></script>
<script type="text/javascript" src="http://webplayer.unity3d.com/download_webplayer-3.x/3.0/uo/UnityObject2.js"></script>
<script type="text/javascript">
	<!-- 
	var config = {
		width: 640,
		height: 360,
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
		u.initPlugin(jQuery("#unityPlayer")[0], "/assets/web_player.unity3d");
	});


	-->
</script>

<style type="text/css">
<!--
div#unityPlayer { cursor: default, height: 360px, width: 640px }
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

and the [source code](https://www.github.com/mjriley/RandomEnemy)
