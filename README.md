# ECX-Haxe-Notes
Write up on basic ECX usage, an ECS for Haxe. Examples are mostly taken from https://github.com/eliasku/ecx-richardlord-asteroids/tree/develop/src/net/richardlord/asteroids. 

This short article is meant to simplify some concepts of ECX in clearer English. **IT IS NOT GUARANTEED TO BE 100% ACCURATE**, but I've tried my best given my understanding of how it works, and hopefully even if things are wrong the general idea is still functional.

# Where get? Where docs? Where example?
- Use this fork of ECX which works with Haxe 4+: https://github.com/sh-dave/ecx

- Use this page for the API. NOTE: It is missing some **very important functions, specifically get(), set(), and create()** which are provided by AutoComp and basically necessary to actually use ECX: https://eliasku.github.io/ecx/api-minimal/

- Use this game as a reference while reading through this: https://github.com/eliasku/ecx-richardlord-asteroids/tree/develop/src/net/richardlord/asteroids

# Macro Spam
ECX uses a lot of macros that break Haxe's rules. It contains several instances of automated variable assignment and generates most things for you. Wire<T> and Family<T> are great examples of this in action.
	
# What is a World?
Contains all the assigned Services (Systems, Components, Services) that have been assigned to it from the WorldConfig it was instantiated with. More than one World should be possible.

# What is a Component?
Components are (mostly) data classes, although they are not excluded from having other functions. Don't make them yourself, just use AutoComp.

# What is AutoComp?
Builds components for you. Function new(){} should apparently be empty, and instead member variables should have default assignments or a specific setup function (appropriately named setup(){} ) should be used. Example of a minimal AutoComp class:

```Haxe
package net.richardlord.asteroids.components;

import ecx.AutoComp;
import flash.geom.Point;

class Position extends AutoComp<PositionData> {}

class PositionData {

	public var position:Point;
	public var rotation:Float;

	public function new() {}

	public function setup(x:Float, y:Float, rotation:Float) {
		position = new Point(x, y);
		this.rotation = rotation;
	}
}
```

In the above example, PositionData will not be directly used by you, and is only used internally by ECX for macro bullshit. Instead, you will access components/the component list called Position when actually working with this component type.

# What is a Wire?
An automated macro tool that injects Services (anything that extends from Service, which by default includes Systems and Components) into other classes in a convenient fashion. Utility classes that you want to use in your program using ECX should always extend from Service to permit ECX to be aware of it, and allow Wire<T> to function appropriately.
	
To use Wires (or most of ECX, really), the World has to have been loaded with a WorldConfig that added the particular Service. Example here: https://github.com/eliasku/ecx-richardlord-asteroids/blob/develop/src/net/richardlord/asteroids/Main.hx

# How do Wires work with Components?
Wiring in a Component type such as 
```Haxe
var _position:Wire<Position>;
``` 
grants access to that particular component type's component list, and permits you to generate instances of that component by providing the create(entityID) function to it. In addition, creating the component immediately returns it so it can be captured like so:
```Haxe
var position = _position.create(entityID);
```

\_position is immediately available to use in your code without proper instantiation, assuming you have Wired it; a nightmarish macro hellscape handles this for you.

# How do Wires work with other Services, including Systems?
You get access to the class instance's features, that's about it.

# What is a Family?
A Family is basically a selection query that populates a list with Entities matching the query. For example:
```Haxe
var _aoeDebuffSpells:Family<Debuff, AreaOfEffect, Position>;
```
\_aoeDebuffSpells will be some list, populated with all entities that only have the Components Debuff, AreaOfEffect, and Position. Like with Wires, \_aoeDebuffSpells is immediately available to use in your code without proper instantiation, assuming you have Family'd it.

# What is a System?
Systems are where the bulk of the work gets done. Your custom system, say, a Collision System, should extend from System. You'll usually provide several Familys and Wires into each system to accomplish whatever task you're trying to do. 

You specifically try and modify the component data of the entities that have been passed in from Familys.

# How the hell do I actually update the World?
The "official" way is to use SystemRunner from ecx-common: https://github.com/eliasku/ecx-common/blob/develop/src/ecx/common/systems/SystemRunner.hx and execute updateFrame. Alternatively, rip out the update loop specifically and slam it somewhere: 
```Haxe
for(system in world.systems()) {
	@:privateAccess system.update();
	world.invalidate();
}
```
Why does ECX udpate everything by using @:privateAccess? It is a mystery.
