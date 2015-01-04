.. _engineoverview:

Game Engine Overview
=======================================

The "Game Engine" refers to all the core pieces that power the game. We will go through the files in an order that makes sense to trace through what happens when you run the game, and highlight what each piece if responsible for. 

Main.lua: Where it All Begins
########################

Main.lua is Love's constructor. It is where our journey through the engine begins! 

It contains the various `callbacks <https://www.love2d.org/wiki/love#Callbacks>`_ we'll use to handle everything from rendering to keyboard and joystick input. 

For simplicity's sake, Main.lua contains as little code as possible. All callbacks are abstracted either through GameSetup [link here] or Events [link here].

**love.load** is our entry point. It gets called once when the game is run.

.. note:: love.load is called even **before the game window is initialized**. Avoid using any calls to love.graphics until the game window is created with love.window.setMode in GameSetup.lua, otherwise the game will not run.

GameSetup handles all of our initialization that happens in love.load. It also has implementations for the main render and update functions that get called in **love.draw** and **love.update** respectively. 

**love.keypressed**, as well as mouse and joystick callbacks are all handled through Events. Check its class-specific page to learn about how keyboard, mouse and joystick input is handled in the engine. 

**love.run** shows you the timeline of how things are called. It is essentially the first function that is called, which then calls love.load and sets up the other callbacks.

GameSetup: The Engine's Seed
############################

I call it the seed because GameSetup is like this core that's always running and everything else sprouts (gets called) from it. This setup makes the engine very flexible. *You could code an entirely different game inside this engine without screwing up anything else!*


