# ECX-Haxe-Notes
Write up on basic ECX usage, an ECS for Haxe. Examples are mostly taken from https://github.com/eliasku/ecx-richardlord-asteroids/tree/develop/src/net/richardlord/asteroids. 

This short article is meant to simplify some concepts of ECX in clearer English. **IT IS NOT GUARANTEED TO BE 100% ACCURATE**, but I've tried my best given my understanding of how it works, and hopefully even if things are wrong the general idea is still functional.

# Macro Spam
ECX uses a lot of macros that break Haxe's rules. It contains several instances of automated variable assignment and generates most things for you. Wire<T> and Family<T> are great examples of this in action.

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
# What is a Wire?
An automated macro tool that injects Services (anything that extends from Service, which by default includes Systems and Components) into other classes in a convenient fashion. Utility classes that you want to use in your program using ECX should always extend from Service to permit ECX to be aware of it, and allow Wire<T> to function appropriately.

# How do Wires work with Components?
Wiring in a Component type such as 
```Haxe
var _bullet:Wire<Bullet>
``` 
grants access to that particular component type's component list, and permits you to generate instances of that component by providing the create(entityID) function. \_bullet is immediately available to use in your code without proper instantiation; a nightmarish macro hellscape handles this for you.

# How do Wires work with other Services, including Systems?
You get access to the class' features, that's about it.

# What is a Family?
A Family is basically a selection query that populates a list with Entities matching the query. For example:
```Haxe
var _aoeDebuffSpells:Family<Debuff, AreaOfEffect, Position>
```
\_aoeDebuffSpells will be some list, populated with all entities that only have the Components Debuff, AreaOfEffect, and Position. Like with Wires, \_aoeDebuffSpells is immediately available to use in your code without proper instantiation.

# What is a System?
Operates on data provided to it. You'll usually provide several Familys and Wires into each system to accomplish whatever task you're trying to accomplish.







