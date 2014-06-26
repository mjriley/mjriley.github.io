---
layout: post
title: "Improving Unit Tests"
modified: 2014-06-26 11:05:14 -0700
category: [Unit Testing]
tags: [unit.testing loldps javascript]
image:
  feature: 
  credit: 
  creditlink: 
comments: 
share: 
---
I'd like to demonstrate the code I've been working with recently that has led me to change the way I will do unit testing going forward. Working on my League of Legends DPS site, I had to create a class to resolve damage between an attacker and a defender. Resolving damage is a complicated process; you need to calculate:

* the damage for a normal hit
* the damage for a critical strike
* the reduction for the defender's armor
* the modified armor due to effects like armor penetration
* and so on...

Dealing only with armor and armor penetration, my calculator looked something like this:

~~~ javascript
var damageCalculator = function(attack, defender) {
	var armor = getDefenderArmor(defender);
	var multiplier;
	armor = computeArmorAfterPercentPenetration(armor, attack.arpen_percent);
	armor = computeArmorAfterFlatPenetration(armor, attack.arpen);
	multiplier = computeArmorMultiplier(armor);
	return attack.damage * multiplier;
}
~~~

where this function was defined in a module, and all the sub-functions (`computeArmorMultiplier`, etc) were private to that module. My unit tests looked something like this:

~~~ javascript
describe("damageCalculator", function() {
	it ("should deal true damage with no armor", function() {...})
	it ("should reduce damage by armor", function() {...})

	describe("percent armor penetration", function() {
		it ("should reduce armor by a percent", function() {...})
	}

	describe("flat armor penetration", function() {
		it ("should reduce armor by a flat amount", function() {...})
		it ("should not reduce armor below 0", function() {...})
		it ("should apply after percent penetration", function() {...})
	}
})
~~~

The above tests are actually testing at two different levels of abstraction! They are testing the basic functionality of armor, percent penetration, and flat penetration, but they are also testing the *interaction* between percent penetration and flat penetration. The above doesn't look too bad, but historically it has deterred me from writing the tests, and especially from maintaining them. When enough conditions are present, it generally seems to culminate in writing something like `it ("should compute the damage correctly")`, which tries to ensure all the conditions are tested simultaneously, but which ultimately doesn't tell anyone anything.

I believe that nested describe 'sub-features' are a bad code smell if that test file also contains features at a higher level. It should signal you to create new classes/modules. I believe the following is a better version of the initial code:

~~~ javascript
var damageCalculator = function(attack, defender) {
	var armor = getDefenderArmor(defender);
	var multiplier;
	armor = damageCalculations.modifyArmorByPercentPenetration(armor, attack.arpen_percent);
	armor = damageCalculations.modifyArmorByFlatPenetration(armor, attack.arpen);
	multiplier = damageCalculations.getArmorMultiplier(armor);
	return attack.damage * multiplier;
}
~~~

Each of these functions will now be publicly exposed in the damageCalculations module, which allows tests to be written strictly to their individual functionalities. The `damageCalculator` tests could now look like:

~~~ javascript
describe("damageCalculator", function() {
	var percentPenetration;
	var flatPenetration;
	
	beforeEach(function() {
		percentPenetration = sinon.spy(damageCalculation, "modifyArmorByPercentPenetration");
		flatPenetration = sinon.spy(damageCalculation, "modifyArmorByFlatPenetration");
	}

	afterEach(function() {
		flatPenetration.restore();
		percentPenetration.restore();
	}
	
	it ("should first apply percentage armor penetration, and then flat armor penetration", function() {
		var attack = ...; var defender = ...;
		damageCalculator(attack, defender);
		flatPenetration.calledAfter(percentPenetration).should.equal(true);
	})
})
~~~

I believe my reservations about this style of code are tied to class proliferation and encapsulation. Because these functions are only ever meant to be called by the calculator class, why should they be exposed to everyone else? And obviously, the number of 'classes' rises if I extract the local methods out into their own individual files.

However, I don't think these arguments are valid. The functions I'm exposing don't change state, so there's no reason they couldn't be used in a different piece of code. With good code organization (i.e. sticking damageCalculation classes in a damageCalculation folder), it should be obvious that, while public, they have a specific intent; if someone else feels that intent aligns with whatever they're doing, re-use is ideal. In terms of proliferation, it's still the same amount of code, just shifted into different groupings. Again, provided it is housed in its own folder, there shouldn't be any extra mental effort required to understand 4 local methods in the same file vs 4 public functions in 4 separate files.
