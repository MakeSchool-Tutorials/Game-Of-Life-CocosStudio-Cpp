---
title: Get Started with SpriteBuilder!
slug: get-started-with-spritebuilder
---       

Installation
============

We are going to use a tool called SpriteBuilder to make this game. When
we are done, the game will look like this:

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+SpriteBuilder+Tutorial/GOF-GridComplete.png)

Go to [the Mac App
Store](https://itunes.apple.com/us/app/spritebuilder/id784912885?mt=12)
and download the latest version of SpriteBuilder. It's free! Drag
SpriteBuilder onto your dock from your Applications folder so that you
can easily access it.

Open SpriteBuilder.

SpriteBuilder Basics
====================

SpriteBuilder's main goal is to provide a tool similar to Xcode's
Storyboard but for Cocos2d games.

SpriteBuilder is a visual editor that allows you to rapidly create
Cocos2d games. It enables you to create user interfaces, gameplay
scenes, and levels by dragging different components to different
*interface files* and arranging their positions. This can save a lot of
time compared to positioning every element on the screen in code.

Next to this core functionality SpriteBuilder includes tools to manage
your assets, create animations, audio effects, and particle effects. We
will get to these advanced features towards the end of this tutorial.

SpriteBuilder Workflow
======================

When you use SpriteBuilder for your game, you start by creating a new
SpriteBuilder project instead of an Xcode project. When creating a
SpriteBuilder project, SpriteBuilder will create and maintain an
embedded Xcode project for you.

Inside the SpriteBuilder project you will organize all the resources and
assets for your game. You will create interface files for the different
scenes in your game. The interface files are called .ccb files, named
after SpriteBuilder's predecessor CocosBuilder. SpriteBuilder also
allows you to create *code connections*. With code connections you can
create links between .ccb files and Objective-C classes. This means you
can add behavior to your game's objects in SpriteBuilder *and* in code -
we will discuss this concept in depth later.

In general your workflow with SpriteBuilder will look like this:

-   Create a new project in SpriteBuilder
-   Add images and other resources to your SpriteBuilder project
-   Create multiple .ccb files for the different scenes and objects in
    your game
-   Add code connections to extend the behavior of these scenes and
    objects
-   *Publish* your project in SpriteBuilder. This will update the Xcode
    project that is linked to your SpriteBuilder project
-   Run your game from Xcode

When you run your game from Xcode, a component called *CCBReader* will
read all the .ccb files from your SpriteBuilder project and create
Cocos2d Scenes and Nodes (the basic classes in Cocos2D) out of them.
Here's a short visualization of how SpriteBuilder and Xcode projects
work together:

![image](https://s3.amazonaws.com/mgwu-misc/Spritebuilder+Tutorial/spritebuilder_publishing.png)

Read on to learn how to create your first project!