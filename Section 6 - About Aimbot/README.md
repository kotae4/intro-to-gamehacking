# Aimbot #

Knowing where your enemies are obviously confers a huge advantage, but what if you can also remove the need to aim at them? An aimbot is a type of cheat that automatically moves your cursor to the enemy. You can also add rules for selecting which enemy to aim at such as closest by distance, closest by angle, only enemies within a certain angle, etc.

## Requirements for Aimbot (Internal) ##

1. A game function you can hook that gets called every frame.
2. The address of the local player instance.
3. The address of the player list or entity list.
	<ol type="a">
	<li>If entity list: figuring out enough of the entity struct to be able to determine what <em>type</em> of entity it is.</li>
	</ol>
4. Figuring out enough of the player / entity struct to read its position and angle (pitch and yaw) as well as the spatial offset to where bullets come from (the eyeheight in some FPS games).
5. A function to calculate the angle you need to rotate by in order to be looking at the enemy (this is the math part).
6. A way to rotate your character.
7. (optional) The engine's traceline / raycast function to determine if any collidable object is in the path between you and the target.

You may notice the first four requirements are shared by ESP cheats, so if you're building both an ESP and aimbot there isn't as much work as it may seem.

## Calculating the Aim Angle ##

As mentioned in the requirements, this is **the math part**. If you read the ESP section you should already know math is not my strong suit. That said, I feel like understanding this part is slightly easier than understanding perspective projection. However, I still find myself having to look up how other people do it when the formula I'm used to doesn't work for whatever reason. Again, I just build up a collection of formulas and do a lot of trial-and-error testing until I find the one that works with the game / engine I'm working on.<br>

The core idea is that your player character is at a certain position and your target character is at another position. You need to determine at what angle your character must face to be looking at the target character.<br>


### TO-DO ###
Move this to a new 'General Game Reversing' subsection in the Information Gathering section, and rewrite it to make it fit:<br>
Something I forgot to mention in the ESP section is that a player's position may not necessarily be part of the player struct. Modern game engines abstract nearly everything, even to the point that there is no 'player' struct. So what you may find is the root 'object' associated with the player and then a bunch of attached 'components' that define that object as a 'player object'. One of those components may be something like a 'Health' component that stores the player's health, armor, and handles everything to do with taking damage. Another component may be the 'Transform' component that stores the position and rotation. One of the challenges with reverse engineering modern games is peeling back these complex layers of abstraction. Since every engine is different, I can't offer more advice other than "be aware that abstraction exists".

## Facing the Target ##

Once you know how much you need to rotate your character, you then need to figure out how to actually rotate your character with code.<br>
There are a few approaches:
1. Use mouseeventf / SendInput to programmatically move your mouse.
2. Use an engine function to rotate your character.
3. Set your character's angles directly (essentially what the engine function would be doing).

The first approach should be the least desirable: it relies on winapi functions and the movement will be marked as 'injected' so it could easily be ignored by the game or detected by any anticheat. If you do go this approach, you'll need to refer to the msdn page about mouse coordinates. There's a translation process you'll need to do.<br>
The second approach is good but requires finding such a function and if you've got the player's rotation fields anyway then it'd be best to just go with the third approach and save yourself the tiny overhead that comes with function calls. That said, there are some cases where this approach will make sense, like if there's too much abstraction for you to make sense of and you just want to get it working. (can you tell I'm speaking from experience?)<br>
The third approach is probably the most common and most desirable. You just set your character's angles directly to the required aim angle.

There are probably other approaches but I haven't experimented with them. There's a type of aiming called 'silent aim' but I haven't looked into it too much, I think it just abuses some specific weakness of the game engine where spectators see one thing but the part of the server that runs hit detection sees another thing. Might be wrong.

## (Stop) Aiming Through Walls ##

The first aimbot you'll make will probably lock onto enemies through walls. At some point this will become very annoying. You might try adding filters to only aim at enemies within a certain aim angle, but you know that's only a band-aid solution. So how do you solve this permanently? Physics! Thankfully we don't need to know any actual physics, we just need to find the part of the game / engine that **does** know physics.<br>
The specific part I'm talking about is commonly called the TraceLine / Raycast / Raytrace / Linecast function. Essentially, this function will throw out an invisible laser beam starting from an origin position and traveling along whatever direction for some distance (or infinite distance). If that beam touches any **collidable, physical object** then the function will return that information to you.<br>
Since this is an engine function, specifically a physics engine function, it's extremely difficult to replicate this in an external cheat, so this is often only used in internal cheats (or an external cheat that injects shellcode to call the function, at which point I don't consider it to be 'external').<br>
A tip for finding this function: make note of any part of the game where looking at something will cause a prompt, label, or something else to appear.<br>
Some examples: aiming at a door will cause "Press F to open" to appear, aiming at a teammate or enemy will cause their name to appear above their head or elsewhere on the screen, aiming at a teammate or enemy will cause your crosshairs to change from white to green or red or whatever color.<br>
You'll want to verify that this change occurs as a result of you **aiming** at the thing, not just by being close to it or by reaching some specific game state. If the prompt appears even if you're aiming at the sky then it's likely due to proximity, for example.<br>
Once you've figured out a place where this occurs then you've got the challenge of finding the function call responsible. Maybe try finding what accesses the string or what writes to the color of the crosshairs. Or find whatever function handles the interaction and work backwards. Using the door example, find the function that opens the door when you press F and recognize that the same function, or one of its callers, also probably calls the Raytrace function to determine that you are actually looking at the door when you press the F key. Maybe you'll start with the core input handler itself and work forwards instead of backwards. Or maybe the game / engine you're working with is open source and you can just look for it in the source code and find any hardcoded strings nearby that you can cross-reference.<br>

## Pixel Scanners ##

This is usually a type of external aimbot, but sometimes it's done internally too. Basically it takes a screenshot of the game and looks for patterns of pixels associated with player characters and then aims there. Obviously, this is **extremely** inefficient and error-prone (what happens when you turn post-processing effects on? Suddenly the pixels don't match anymore). Not only is it computationally expensive to run **any** image processing (even just iterating pixels and performing comparisons), it's also quite expensive to capture these screenshots in the first place, so this method should be avoided at all costs.<br>
Furthermore, even just gathering the patterns of pixels to look for requires a lot of manual labor because of things like lighting and distance.<br>
Still, the benefit of this is similar to that of chams: you don't need to do any reverse engineering or write any new code to support additional games.

## Summary ##

Aimbots don't require much more work than a simple ESP, and, just like with ESPs, we can leverage the game's own functions to reduce our work.

### TO-DO ###

Add more stuff and things. Seriously.