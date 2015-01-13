.. _engineoverview:

Game Engine Overview
=======================================

The "Game Engine" refers to all the core pieces that power the game. We will go through the files in an order that makes sense to trace through what happens when you run the game, and highlight what each piece if responsible for. 

Main.lua: Where it All Begins
#############################

Main.lua is Love's constructor. It is where our journey through the engine begins! 

It contains the various `callbacks <https://www.love2d.org/wiki/love#Callbacks>`_ we'll use to handle everything from rendering to keyboard and joystick input. 

For simplicity's sake, Main.lua contains as little code as possible. All callbacks are abstracted either through GameSetup [link here] or :ref:`Events` .

**love.load** is our entry point. It gets called once when the game is run.

.. note:: love.load is called even **before the game window is initialized**. Avoid using any calls to love.graphics until the game window is created with love.window.setMode in GameSetup.lua, otherwise the game will not run.

GameSetup handles all of our initialization that happens in love.load. It also has implementations for the main render and update functions that get called in **love.draw** and **love.update** respectively. 

**love.keypressed**, as well as mouse and joystick callbacks are all handled through :ref:`Events`. Check its class-specific page to learn about how keyboard, mouse and joystick input is handled in the engine. 

**love.run** shows you the timeline of how things are called. It is essentially the first function that is called, which then calls love.load and sets up the other callbacks.

GameSetup: The Engine's Seed
############################

I call it the seed because GameSetup is like this core that's always running and everything else sprouts (gets called) from it. This setup makes the engine very flexible. *You could code an entirely different game inside this engine without screwing up anything else!*

GameSetup is responsible for the following tasks:

* Loading config files and initializing the game window on startup (GameSetup:init)
* Loading game modes and giving them access to the update and render callbacks in main.lua 

What makes this system so versatile is the GameMode [link here] system. GameSetup is responsible **for nothing other than setting up and implementing the GameMode system.** It is up to each individual game mode to initialize and run the pieces it needs of the engine. 

Think of it like a component-based system. A bare game mode will be nothing more than a black sceen. If you want to render anything, just import the RenderingSystem [link here]! Want some sound? We have the SoundSystem [link here]. What about some physics (Box2d) ? You can import that too! All these difference pieces are self-contained. This setup was meant to keep the code clean and easy to debug. As a bonus, this also means it's very easy to switch out to a modified rendering system or completely different physics, or add new components! 

.. note:: Move or Die actually started as just that: a new game mode within Concerned Joe. That's why the mode that runs it is called PartyMode.lua, in reference to it originally being the "Party Mode" of Concerned Joe. While the two seperate games were able to co-exist on the same engine without interference, there were occasions when we had to hardcode something in. This is where you'll see the global variable _PARTY in some places in the code. 

So Game Modes really are where all the meat is. So let's take a look at them next!

Game Modes: The Heart
#####################

Game Modes are self-contained states. They must clean up everything they do. I call it the heart of the game because looking at it bare allows you to see everthing that's moving in the engine. 

GameMode.lua [link here] itself is something like a virtual class with empty functions for **init**, **cleanup**, **update** and **render**. Those are the four functions you need to overwrite when you create a new mode. The other functions provide useful methods for GameSetup's implementation. 

Let's take a look at a simple Game Mode that implements the engine's main components::

	local TestMode = {};
	local GameMode = require("Engine/GameMode");
	TestMode = GameMode:create("BasicTest");

	local hash = require("Engine/SpatialHash");
	local rendering = require("Engine/RenderingSystem");
	local soundsys = require("Engine/SoundSystem");

	function TestMode:init()
	end

	function TestMode:cleanup()
	end

	function TestMode:update(dt)
		hash:Update();
		_world:update(dt * 2);
		rendering:update(dt);
		soundsys:update(dt);
	end

	function TestMode:render()
		rendering:render();
		soundsys:render();
	end

	return TestMode;

This is all the code you need to run all the basic functionality of the game (aside from game logic). Let's go through this and see exactly what's going on::

	local TestMode = {};
	
This line just initializes our class object so that this file can be loaded as a Lua module. This can be called anything as long as the function are attached to it and it's returned at the end of the file. For clarity, it should be similar to the name of the class/file::

	local GameMode = require("Engine/GameMode");
	TestMode = GameMode:create("BasicTest");
	
Here we load the GameMode class and initialize our new game mode. We give it the name "BasicTest". This is the name we'll use to load it later in GameSetup::

	local hash = require("Engine/SpatialHash");
	local rendering = require("Engine/RenderingSystem");
	local soundsys = require("Engine/SoundSystem");
	
We include 3 out of the 4 systems we're using. SpatialHash [link here] is a special class that takes care of keeping track of where things are on screen. It's mainly used in RenderingSystem [link here] which is our second class. As the name implies, it takes care of rendering everything on screen. SoundSystem [link here] takes care of audio. The fourth system that isn't mentioned here is Box2d, our physics system. The physics `world <https://www.love2d.org/wiki/World>`_ object *_world* is a global that can be accessed without needing to import anything::

	function TestMode:init()
	end

	function TestMode:cleanup()
	end
	
Here we have our **init** and **cleanup** functions. They're empty because we're not creating or altering anything that needs to be destroyed or reset. It's good practice to always define these, otherwise the default functions will be used (which will just print an example message to the console)::

	function TestMode:update(dt)
		hash:Update();
		_world:update(dt * 2);
		rendering:update(dt);
		soundsys:update(dt);
	end
	
Here is where all 4 of our systems are being updated. "dt" is the delta time (the time that has passed since the last frame). Box2d's timestep is twice that of the rest of the game. This is purely a game design choice (the physics would feel too floaty otherwise). And lastly::

	function TestMode:render()
		rendering:render();
		soundsys:render();
	end
	
Only the Rendering and Sound systems need a render function. It's obvious why the Rendering system would need it. For the Sound, it is sometimes useful to toggle a sort of debug drawing for static sounds, to see their position and range, otherwise the sound system deosn't really render much.

Conclusion
##########

That's it! You now know the gist of how the engine is structured. Of course there's far more to be said in how the engine works, but this at least covers enough of the basics so that you have a generla sense of how everything connects and know where to look, where to insert your own code etc.. 

It is strongly recommended to read about the GameObject [link here] class next, because there isn't much that can be done in the engine without it. Then looking into the four systems mentioned above. 