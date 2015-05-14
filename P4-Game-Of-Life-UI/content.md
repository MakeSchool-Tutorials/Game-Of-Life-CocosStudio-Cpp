---
title: Create the Game of Life's User Interface
slug: game-of-life-ui
---       

Now it's time to create the interface for the game in Cocos Studio. Once you are done with this chapter the UI of the game will look like this:

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/finalUI.png)

Getting Started
===============

Get started by downloading our [art
pack](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/Assets.zip)
for this game.

After you have unzipped it, drag the *Assets* folder into your
Cocos Studio project onto the Resources Browser in left panel. Alternatively you can right click in the Resources Browser and *Import Resources...*.
	
![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/rightClickImport.png)

After importing the assets, your Resources Browser should look like this

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/assetsExpanded.png)

Now open *MainScene.csd* and highlight Default in the timeline.
Hit the delete key - you should end up with a black screen. You just deleted the default background image, but note that you cannot delete the root Scene. 

In the Resources Browser in the left panel, find *HelloWorld.png* and delete it - you don't need it anymore.

Add a Background Image
======================

Drag *background.png* to the stage of *MainScene.csd*. We want to center the background in the scene. To do that, we'll set the *anchor point* and *position* properties of the background image.

The *anchor point* is the point inside that the node that will be moved to whatever *position* coordinates you specify. It's also the point around which the object will be rotated when you modify the *rotation* property.

First, ensure that the anchor point is set to (0.5, 0.5). The anchor point is a coordinate inside the object - it's expressed in percentages from 0 to 1.  In this case, (0.5, 0.5) means (50%, 50%) which will place the anchor point in the middle of the background. 

Now set the background position to be expressed in *%Relative percentage of parent container* by clicking the dropdown that says *Px*. Then set the values to (50, 50). This will set the background's position to (50, 50) of the parent node. By expressing the position of the background in percentages instead of absolute coordinates we make our layout more flexible. That's because, even if the parent node's size changes, the background will remain the middle.

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/positionRelative.png)

Create Layout Panels
====================

To help ensure that our UI resizes dynamically for various screen resolutions, we're going to use two *Panels* which are a type of *Container Node*.  We will have a left panel for the play button, pause button, labels and microscope, and a right panel that will hold the grid.  

Drag a Panel container from the Objects Browser in the left panel. Set the anchor point to (0, 0.5), the position to *%Relative percentage of parent container* and the postion to (0, 50).  This will pin the panel to the middle left of the screen.  Set the size to *%Relative percentage of parent container* and (20, 100). Finally, change the name to *leftPanel*. In the code we'll refer to the objects created in Cocos Studio by their names, so it's important that we give them good ones.

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/leftPanelSettings.png)

Now we'll create the right panel. Drag another Panel container into the scene from the Objects Browser. Set the anchor point to (0, 0.5). Set the position to *%Relative percentage of parent container* and (20, 50). Set the size to *%Relative percentage of parent container* and (80, 100). Change the name to *rightPanel*.

So now your timeline should look like this:

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/panelsSetup.png)

Creating a Grid
===============

We will create the grid in a separate .csd file because it will be linked to a custom class later on. Create a new .csd file *File --> New File* of type *Node* and call it *Grid*.

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/newGrid.png)

Find Grid.png in the Resource Browser and drag it into the your newly created Node. Set the position to (0, 0) to center it.

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/gridImage.png)

**Make sure to save your newly created file (cmd + s) or your grid will not display correctly later on!**

Add the Grid to the MainScene
=============================

Now open *MainScene.csd* again by double clicking on it. 

Drag *Grid.csd* onto the stage. This will add the grid to the
MainScene. Rename it to gridNode. In the timeline, drag gridNode to become a child of rightPanel. 

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/gridNodeChild.png)

Set the position of gridNode to (50%, 50%).

Game UI
====================

Great! Now it is time to set up your game's UI! We're going to add the play and pause buttons, the game labels and the microscope sprite to the scene.  All of these will go on the left side of the screen, so make sure they're children of leftPanel.

Drag two buttons (you can find them in the Widgets section of the Objects Browser) into the scene. Name one *btnPlay* and the other *btnPause*. Make sure they both have an anchor point of (0.5, 0.5).  Set the size of both to (140px, 76px). Position btnPlay at (50%, 88%) and btnPause at (50%, 75%). Delete the default "Button" text for both.

Double click the *Normal State* property for btnPlay. Navigate to and select play.png. 

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/btnPlayNavigation.png)

For *Press State* select play-pressed.png. Do the same for btnPause with pause.png and pause-pressed.png.

Now go to your resources and add *balloon.png* and *microscope.png* to leftPanel. Set the anchor points of both to (0.5, 0.5). Name the balloon sprite "balloon" without the quotes.  Position balloon at (50%, 52%). Position the microscope at (50%, 20%).  

When you are done it should look like this:

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/gameUIPreLabels.png)

In the next step add four labels as children of the balloon, these will form our scoreboard. To do so, go back to the Objects Browser, find the Widget called *Label* (not *BitmapLabel*), and drag one of them onto the scene. Scroll down to the bottom of the label's properties and click the *Font file* button. Select Courier New Bold.ttf from the Fonts directory. Set the size to 20, and the color to #0D9F00. Copy and paste that label 3 times, then fill out the labels according to the following table.

| Label Name      | Label Text | Position   |
|-----------------|------------|------------|
| populationLabel | Population | (52%, 75%) |
| populationCount | 0          | (52%, 59%) |
| generationLabel | Generation | (52%, 40%) |
| generationCount | 0          | (52%, 23%) |

When you're done it should look like this:

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/labels.png)

Great! Now it's time to set up code connections before we switch to Xcode and you start implementing the actual game. We want to tell Cocos Studio how this UI you've created will interact with the classes and methods we will implement in code.

Set Up Code Connections
=======================

Now we have to tell Cocos Studio which methods to call when our
buttons are touched. Select *btnPlay*, go to the Advanced view (second button on the top of the right panel). Under callback method, select *Touch* in the dropdown, and type *play* in the text box on the right.

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/playCallback.png)

**Repeat this step and set the selector *pause* for the pause button.**

Now save, and publish to Xcode.

Your Xcode project is contained inside the *proj.ios_mac* sub-directory in the directory where your Cocos Studio project lives. If you don't remember where you saved it, search for your project name in Spotlight (magnifying glass at the top right of your Mac's screen).
