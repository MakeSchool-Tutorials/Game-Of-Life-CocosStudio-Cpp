---
title: Finish the Game of Life!
slug: game-of-life-code
---       

Time to code! In this step we are going to hook up the UI we've created
in Cocos Studio with the game logic we're going to code in Xcode.

Open your Xcode project if it isn't already. Your Xcode project is contained inside the *proj.ios_mac* sub-directory in the directory where your Cocos Studio project lives. If you don't remember where you saved it, search for your project name *GameOfLife.xcodeproj* in Spotlight (magnifying glass at the top right of your Mac's screen).


Modify AppDelegate.cpp
======================

First we will modify our AppDelegate class.  The AppDelegate is the class that handles interfacing with the operating system.  For example, `void AppDelegate::applicationDidEnterBackground() ` is called every time your game is backgrounded.

We're going to modify `bool AppDelegate::applicationDidFinishLaunching()`. This method is called the first time your game is loaded, and is where most of our game-specific Cocos2d-x setup occurs. 

Change this line:

	director->getOpenGLView()->setDesignResolutionSize(960, 640, ResolutionPolicy::SHOW_ALL);
	
to look like this:

	director->getOpenGLView()->setDesignResolutionSize(960, 640, ResolutionPolicy::FIXED_HEIGHT);
	
This will make sure that the game won't be letterboxed (black bars appearing on the sides) for the various iPhone resolutions.

Press the play button to see the UI of your game in the iOS simulator!

Create a Creature class
=======================

We're going to implement this game in an object-oriented manner - therefore
Creatures (the things spawning and dying in the grid) will get their own class. Create a new C++ class in Xcode called Creature. Go to File --> New --> File then choose C++ File. 

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/createNewFile.png)

Create a class called *Creature*.

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/createCreature.png)

Check the target for GameOfLife Mac, and make sure you save it in the Classes directory of your project.

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/creatureSaveLocation.png)

When you've created the class, open the Creature header file (*Creature.h*). The first thing you'll notice is that Xcode automatically generated an [#include guard](http://en.wikipedia.org/wiki/Include_guard) for you. It should look something like this:

	#ifndef __GameOfLife__Creature__
	#define __GameOfLife__Creature__

	// Header code goes here

	#endif /* defined(__GameOfLife__Creature__) */

*The code you write in any header file should always be inside the include guards.* 

First, delete `#include <stdio.h>`, we don't need it. Then paste the following code between the include guards. 
    
	#include "cocos2d.h"

	class Creature : public cocos2d::Sprite
	{
	public:
	    
	protected:
	
	};


The first line:

	#include "cocos2d.h"

includes the Cocos2d-x header files into this file, so we can reference the Cocos2d-x classes.  We need that for the following line to work:

	class Creature : public cocos2d::Sprite

This is a C++ class declaration. We're declaring a class called `Creature` that inherits from `cocos2d::Sprite`. In Cocos2d-x (and most game engines) a *Sprite* is a kind class that can display an image.

After `public:` and `protected:` is space for us to declare both public and protected methods and instance variables for the `Creature` class.

Place this code below `public:`:

    CREATE_FUNC(Creature);
    
    bool init() override;

This is some boiler-plate code that you will place in all of your custom classes that inherit from a Cocos2d-x node or node subclass. `CREATE_FUNC()` is a macro that will both declare and implement a `create()` method for your class. In this case, that means we can now create a new `Creature` by calling `Creature::create()`.
	
`init()` is a method that is automatically called by `create()`. `init()` is where you will place your initialization code for the class. Because we're overriding `init()` from a super-class (`cocos2d::Sprite`), we use the `override` keyword to notify the compiler and any humans reading the code. `override` is optional, but is a good practice.

Now, below where you declared `init()` we'll declare the public methods that other classes will use to interface with the `Creature` class. They look like this:

    void setLivingNeighborsCount(int livingNeighborsCount);
    int getLivingNeighborsCount();
    
    void setIsAlive(bool isAlive);
    bool getIsAlive();
   
    
In this case, we're declaring two getter and setter methods, for two properties of the `Creature` class. The first property is `isAlive` which is a boolean value indiciating if the creature is alive or not. The second is `livingNeighborsCount`, an integer indicating how many neighboring cells are occupied by a living creature.

Now let's declare the instance variables for those two properties, below the `protected` keyword:

	protected:
		int livingNeighborsCount;
		bool isAlive;	

They're `protected` because we don't want outside classes to be able to modify their values directly - they'll have to use the publically declared getter and setter methods to do that.

Now let's implement the code for our `Creature` class. Switch to *Creature.cpp*. Right below `#include "Creature.h"` add the following line:

	using namespace cocos2d;
	
This will allow us to use Cocos2d-x classes, like `Sprite`, without having to prepend them with `cocos2d::`. Below that, add this `init` method:

	bool Creature::init()
	{
	    if (! Sprite::initWithFile("Assets/SpriteImages/bubble.png"))
	    {
	        return false;
	    }

	    this->setLivingNeighborsCount(0);
	    this->setIsAlive(false);
	    
	    return true;
	}

This initializer sets the image of the creature to *bubble.png*. It does that by calling the superclass' initializer, `Sprite::initWithFile()`. We're also careful to initialize the values of our properties by calling their setters. We set `livingNeighborsCount` to *0* and `isAlive` to *false*. We return *true* at the end to indicate that initialization was successful. 

Next let's create our getter and setter methods for the `livingNeighborsCount` property:

	void Creature::setLivingNeighborsCount(int livingNeighborsCount)
	{
	    this->livingNeighborsCount = livingNeighborsCount;
	}

	int Creature::getLivingNeighborsCount()
	{
	    return this->livingNeighborsCount;
	}
	
This is the standard way to implement getters and setters in C++, so read it carefully. In `setLivingNeighborsCount` we use `this` to refer to the instance variable that's part of the class `Creature`. That is to distinguish it from the parameter that has the same name: `livingNeighborsCount`. The getter is easy, it simply returns the value of `this->livingNeighborsCount`.

Now implement `setIsAlive` and `getIsAlive` the same way, taking care to remember that `isAlive` is a `bool`, and not an `int`. Once you've done that, add the following line to your `setIsAlive`:

	this->setVisible(isAlive);
	
This calls the superclass method `setVisible` which does just like what it sounds like - makes the `Creature` visible or invisible depending on the value of `isAlive`. That way, when the creature is alive, it will be visible, and when it is dead it will be invisible.

Now we're done implementing `Creature`!

Create a Grid Class
===================

Now let's create the *Grid* class. Create a new C++ class in Xcode following the same steps you used to create the Creature class, but name it `Grid`.

Replace the code in *Grid.h* (inside the include guards) with the following:

	#include "cocos2d.h"
	#include "Creature.h"

	class Grid : public cocos2d::Node
	{
	public:
	    CREATE_FUNC(Grid);
	    
	    bool init() override;
	    
	    void onEnter() override;
	    
	    void evolveStep();
	    
	    int getGenerationCount();
	    
	    int getPopulationCount();
	    
	protected:
	    int generationCount;
	    int populationCount;
	    float cellWidth;
	    float cellHeight;
	    cocos2d::Vector<Creature*> gridArray;
	    
	    void setupGrid();
	    void setupTouchHandling();
	    void updateNeighborCount();
	    void updateCreatures();
	    Creature* creatureForTouchLocation(cocos2d::Vec2 touchLocation);
	    bool isValidIndex(int row, int col);
	    int indexForRowColumn(int row, int col);
	};

First, notice that the `Grid` class inherits from `cocos2d::Node`. `Node` is the main base class for Cocos2d-x objects in your scene - it includes properties like position, contentSize and scale, among others. If you remember, when we created the Grid in Cocos Studio, it was of type Node - that's why we're making it inherit from `Node` in our code also.

Just like in the `Creature` class the first two methods are `CREATE_FUNC()` and `init()`. The next method `onEnter()` will be used to do additional setup. `evolveStep()` is what will get called to trigger a step in the simulation. The last two methods, `getGenerationCount()` and `getPopulationCount` are getters that we'll use to populate the labels in the UI.

Notice that the 5 declarations directly below `protected` are variables, and that the following 7 declarations are methods. When making declarations, it's a good idea to group variables and methods separately to make your code easy to read.

Setup the Grid
===================

Switch over to *Grid.cpp* so that we can begin to implement the grid. Directly under `#include "Grid.h"` type the following:

	using namespace cocos2d;

	const int ROWS = 8;
	const int COLUMNS = 10;

We declare ROWS and COLUMNS to be `const`, as in *constant* because our grid is static, and we don't want the value of either to change. It's common to declare constant variables on all-caps.

Below that type the `init()` method:

	bool Grid::init()
	{
	    if (! Node::init())
	    {
	        return false;
	    }
	    
	    generationCount = 0;
	    populationCount = 0;
	    
	    return true;
	}
	
Here we're initializing our class by calling the superclass initializer `Node::init()`, after which we initialize `generationCount` and `populationCount` to 0.

One important thing to note is that before the `Grid` is fully initialized, certain properties on it cannot yet be accessed - for example the `contentSize` of the grid `Sprite` - because it's not yet loaded. But some of our instance variables, `cellWidth` and `cellHeight` depend on the width and height of the grid! The best way to deal with that is to initialize those variables in `onEnter()`, which is called after the `Node` is fully initialized, and is about to enter the stage. Type the following below `init()`.

	void Grid::onEnter()
	{
	    Node::onEnter();
	    
	    this->setupGrid();
	    
	    this->setupTouchHandling();
	}
	
This is pretty simple - first, because we're overriding the superclass' `onEnter()` (remember the `override` in *Grid.h*?), it's very important that we remember to call the superclass' `onEnter()`, so we do that with `Node::onEnter()`. The next two lines are calling some additional setup methods that we shall implement now. Below `onEnter()`, type the following:

	void Grid::setupGrid()
	{
	    Sprite* gridSprite = this->getChildByName<Sprite*>("grid");
	    cellWidth = gridSprite->getContentSize().width / float(COLUMNS);
	    cellHeight = gridSprite->getContentSize().height / float(ROWS);
	    
	    gridArray.reserve(ROWS * COLUMNS);
	    
	    for (int row = 0; row < ROWS; ++row)
	    {
	        for (int col = 0; col < COLUMNS; ++col)
	        {
	            Creature* creature = Creature::create();
	            
	            creature->setAnchorPoint(Vec2(0.0f, 0.0f));
	            creature->setPosition(cellWidth *  float(col), cellHeight * float(row));
	            
	            gridSprite->addChild(creature);
	            
	            gridArray.pushBack(creature);
	        }
	    }
	}

Here's what these lines do: First we grab a reference to the Grid sprite with `getChildByName`. The name we use is `"grid"`, the same name we gave our grid sprite in Cocos Studio. Next we intialize the values of `cellWidth` and `cellHeight`.

Next, we're going to create a creature for every cell - instead of allocating and deallocating new ones as they live and die, we'll just have a creature exist for each cell that we'll toggle visible and invisible. We're also going to store a reference to each creature in an array called `gridArray` so that we can later access them.

First we allocate enough memory in `gridArray` for each of the creatures that will be stored with `reserve()`. This isn't necessary, but it's a good practice - instead of reallocating memory progressively as we fill it, `gridArray` will have enough memory from the beginning. 

Next, we create a nested for loop the interior of which will be run 80 times (8 rows * 10 columns). For each iteration, it creates a new `Creature` with `Creature::create()`. It sets the anchor point in the bottom left-hand corner, and sets the position. Finally the `Creature` is added as a child of `gridSprite` and added to the `gridArray`.

Add the GridReader
===================

You should recall that when we created *Grid.csd* in Cocos Studio, we assigned it the custom class *Grid*. Cocos2d-x requires that every custom class have an associated *Reader* class which is used to link up the Cocos Studio object with its associated code. Create a new C++ file and call it *GridReader*.

In *GridReader.h*, (between the header guards) paste the following

	#include "cocos2d.h"
	#include "cocostudio/WidgetReader/NodeReader/NodeReader.h"

	class GridReader : public cocostudio::NodeReader
	{
	public:    
	    static GridReader* getInstance();
	    static void purge();
	    cocos2d::Node* createNodeWithFlatBuffers(const flatbuffers::Table* nodeOptions);
	};
	
Then in *GridReader.cpp* add the following code:

	#include "GridReader.h"
	#include "Grid.h"

	using namespace cocos2d;

	static GridReader* _instanceGridReader = nullptr;

	GridReader* GridReader::getInstance()
	{
	    if (!_instanceGridReader)
	    {
	        _instanceGridReader = new GridReader();
	    }
	    return _instanceGridReader;
	}

	void GridReader::purge()
	{
	    CC_SAFE_DELETE(_instanceGridReader);
	}

	Node* GridReader::createNodeWithFlatBuffers(const flatbuffers::Table *nodeOptions)
	{
	    Grid* node = Grid::create();
	    setPropsWithFlatBuffers(node, nodeOptions);
	    return node;
	}
	
This is all boilerplate code - whenever you create a *Reader* class the code will be the exact same, except you will replace all instances of `Grid` with `YourClassName`.

Finally, we have to register the `GridReader` with `CSLoader`. `CSLoader` is the class that reads the binary files exported by Cocos Studio to create instances of the objects in code. So by registering `GridReader` with it, we're saying, "hey, when you encounter an object with the custom class `Grid`, use this `GridReader` to create that object.

In *HelloWorldScene.cpp*, *before* this line of code:

	auto rootNode = CSLoader::createNode("MainScene.csb");

add the following:

	CSLoader* instance = CSLoader::getInstance();
	// Be very careful to do GridReader::getInstance, not GridReader::getInstance() which will crash
	instance->registReaderObject("GridReader", (ObjectFactory::Instance) GridReader::getInstance);
	
Finally, at the top of *HelloWorldScene.cpp* next to the other includes, add `#include "GridReader.h"`.

Test it!
===================

Time to test the code! In your `onEnter()`, comment out `this->setupTouchHandling()` by adding two backslashes `//` in front of it. We haven't yet implemented `setupTouchHandling()` so leaving that in would create a compilation error. Next, in `setupGrid()`, add the following line after `gridArray.pushBack(creature);`

	creature->setIsAlive(true);

This will make the creatures visible. Run the game! You should see something like this:

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+Cocos+Studio+Tutorial/gridTest.png)

**Undo the changes you made in this section before moving on.**
	
Add Touch Handling
===================

Time to add touch handling to the grid. Open *Grid.cpp*. Below `setupGrid`, add the following:

	void Grid::setupTouchHandling()
	{
	    auto touchListener = EventListenerTouchOneByOne::create();
	    
	    touchListener->onTouchBegan = [&](Touch* touch, Event* event)
	    {
	        Sprite* gridSprite = this->getChildByName<Sprite*>("grid");
	        
	        Vec2 gridTouchLocation = gridSprite->convertTouchToNodeSpace(touch);
	        
	        Creature* touchedCreature = this->creatureForTouchLocation(gridTouchLocation);
	        
	        if (touchedCreature)
	        {
	            touchedCreature->setIsAlive(!touchedCreature->getIsAlive());
	        }
	        
	        return true;
	    };
	    
	    this->getEventDispatcher()->addEventListenerWithSceneGraphPriority(touchListener, this);
	}

First we create an `EventListenerTouchOneByOne`. This *event listener* is more specifically a *one-by-one touch listener* - that is, it will listen for touches and tell us about them, one at a time. Next we'll tell the `touchListener` the code we want to execute when the touch *begins*. There's other touch states too, like *moved*, *ended* and *cancelled*, but we won't need to bother with those now.

	touchListener->onTouchBegan = [&](Touch* touch, Event* event)
	{
	
	};
	
This chunk of code takes advantage of a new feature in C++11 called [lambdas](http://en.wikipedia.org/wiki/Anonymous_function#C.2B.2B_.28since_C.2B.2B11.29). Every time there is a touch on the screen, the code inside the block delimited by curly braces`{ }` is executed. This is nice because we're able to write the code right here, instead of having to create a new method for touch handling. Every time this block of code is called, a `Touch*` called `touch` and an `Event*` called `event` are passed in as parameters.

Inside the block the first thing we do is get a reference to the grid sprite, assigning it to `gridSprite`. We use that reference in the next line to convert the touch from *world space* to *node space*. That is, we convert the coordinates from screen coordinates to coordinates inside the grid. Next, we use the grid coordinates with the not-yet-coded `creatureForTouchLocation()` method to figure out which creature (if any) was touched. If there was a touched creature, we flip the value of its `isAlive` property. Finally, we return `true`, to indicate to the game engine that the touch was consumed, and shouldn't be sent to any other listeners.

Then, after the block of code, we add the newly-created `touchListener` to the `EventDispatcher` so that it starts receiving touch events.

Now it's time to code the `creatureForTouchLocation()` method that's called in the `onTouchBegan` block. It should take a position `Vec2` as input, and use it to determine if there's a `Creature` there, and if there is, return it. It looks like this:

	Creature* Grid::creatureForTouchLocation(Vec2 touchLocation)
	{
	    if (touchLocation.x < 0.0f || touchLocation.y < 0.0f)
	    {
	        return nullptr;
	    }
	    
	    int row = touchLocation.y / cellHeight;
	    int col = touchLocation.x / cellWidth;
	    
	    if (this->isValidIndex(row, col))
	    {
	        return gridArray.at(this->indexForRowColumn(row, col));
	    }
	    else
	    {
	        return nullptr;
	    }
	}

First we check if either the x-coordinate or y-coordinate of the touch is `< 0.0f` . If either of those are true, the touch is off the grid, so we return a null pointer. Next we figure out the row and column of the touch based on the cell heights and widths. If the row and column in question are valid `isValidIndex(row, col)` then we return the creature at that index in the `gridArray`, using `indexForRowColumn(row, col)`. Otherwise we return a null pointer again.

This method uses two more uncoded methods, so let's make those. Fortunately, they're simple. Here's `isValidIndex(row, column)`. It should return true if the parameters passed for row and column are within the amount of rows and columns we have:

	bool Grid::isValidIndex(int row, int col)
	{
	    return (row >= 0 && row < ROWS) && (col >= 0 && col < COLUMNS);
	}
	
Next, here's `indexForRowColumn(row, col)`. This takes a row and column as parameters, and returns the integer index for that `Creature` in our one-dimensional `gridArray` array.

	int Grid::indexForRowColumn(int row, int col)
	{
	    return row * COLUMNS + col;
	}
	
Run your game and try tapping on the grid. You should see creatures
coming to life and dying where you tap!

Set Up HelloWorldScene
======================

In *HelloWorldScene.cpp* we will:

-   create a timer that will call the `evolveStep()` method on the grid and
    update the stats and gamestate labels (generation, alive/dead creatures)
-   implement a `play` method that starts the timer
-   implement a `pause` method that stops the timer

But first, go to *HelloWorldScene.h*, and add the following after the `public:` method declarations:

	private:
	    Grid* grid;
	    cocos2d::ui::Text* populationCount;
	    cocos2d::ui::Text* generationCount;
	    
	    void play(Ref* pSender, cocos2d::ui::Widget::TouchEventType type);
	    void pause(Ref* pSender, cocos2d::ui::Widget::TouchEventType type);
	    void step(float dt);

Here we delare three instance variables, `grid`, `populationCount` and `generationCount`. We also declare three methods, `play`, `pause` and `step`. So that the compiler can see will be happy, add these to the `#include` at the top:

	#include "ui/CocosGUI.h"
	#include "Grid.h"
	    
Now, go to *HelloWorldScene.cpp*

Directly below this line:

	auto rootNode = CSLoader::createNode("MainScene.csb");
	
And above this line:

	this->addChild(rootNode);
	
Add the following:

    auto leftPanel = rootNode->getChildByName("leftPanel");
    auto rightPanel = rootNode->getChildByName("rightPanel");
    
    grid = rightPanel->getChildByName<Grid*>("gridNode");
    
    auto balloon = leftPanel->getChildByName("balloon");
    generationCount = balloon->getChildByName<cocos2d::ui::Text*>("generationCount");
    populationCount = balloon->getChildByName<cocos2d::ui::Text*>("populationCount");
    
    cocos2d::ui::Button* playButton = leftPanel->getChildByName<cocos2d::ui::Button*>("btnPlay");
    cocos2d::ui::Button* pauseButton = leftPanel->getChildByName<cocos2d::ui::Button*>("btnPause");
    
    playButton->addTouchEventListener(CC_CALLBACK_2(HelloWorld::play, this));
    pauseButton->addTouchEventListener(CC_CALLBACK_2(HelloWorld::pause, this));

Most of this code is just grabbing references to the various objects we created in Cocos Studio. We store a reference to the grid in the `grid` instance variable. We assign the `generationCount` and `populationCount` labels to their respective instance variables.

Then we grab references to the `playButton` and `pauseButton` and tell them that, upon being clicked, they should call the `play` and `pause` methods.

Let's implement those methods:

	void HelloWorld::play(Ref* pSender, ui::Widget::TouchEventType type)
	{
	    this->schedule(CC_SCHEDULE_SELECTOR(HelloWorld::step), 0.5f);
	}

	void HelloWorld::pause(Ref* pSender, ui::Widget::TouchEventType type)
	{
	    this->unschedule(CC_SCHEDULE_SELECTOR(HelloWorld::step));
	}

These use the Cocos2d-x scheduler to `schedule` and `unschedule` the `step` method, telling it to call it every half second. So lets code `step`:

	void HelloWorld::step(float dt)
	{
	    grid->evolveStep();
	    
	    generationCount->setString(std::to_string(grid->getGenerationCount()));
	    populationCount->setString(std::to_string(grid->getPopulationCount()));
	}

That's it for `HelloWorld` so let's finish up the project by coding the last remaining methods in *Grid.cpp*.

Grid Accessors
======================

`Grid` has two public properties, `populationCount` and `generationCount`. We haven't yet coded the accessors, but fortunately they're easy!

	int Grid::getPopulationCount()
	{
	    return populationCount;
	}

	int Grid::getGenerationCount()
	{
	    return generationCount;
	}

Evolve!
=======

Now for the tricky part. We need to implement the `evolveStep` method in
*Grid.cpp*.

According to the rules of the Game of Life, we need to count how many
live neighbors every cell has every step. If it has 0, 1, 4 or more live neighbors
the `Creature` on that cell dies or stays dead. If it has exactly 3 neighbors and it is dead, it comes to life!

So we need to go through every Creature, count the number of live
neighbors it has, and update whether it is alive or dead.

Fill your `evolveStep()` method with the following code:

    //update each Creature's neighbor count
    this->updateNeighborCount();
    
    //update each Creature's state
    this->updateCreatures();
    
    //update the generation so the label's text will display the correct generation
    generationCount++;

Once we fill out the `updateNeighborCount()` and `updateCreatures()` methods, we're
done!

Declare `updateNeighborCount()` in your *Grid.cpp*. It takes no parameters and has no return type.  Fill it with the following code:

    for (int row = 0; row < ROWS; ++row)
    {
        for (int col = 0; col < COLUMNS; ++col)
        {
            int currentCreatureIndex = this->indexForRowColumn(row, col);
            
            Creature* currentCreature = gridArray.at(currentCreatureIndex);
            currentCreature->setLivingNeighborsCount(0);
            
            for (int nRow = row - 1; nRow <= row + 1; ++nRow)
            {
                for (int nCol = col - 1; nCol <= col + 1; ++nCol)
                {
                    bool indexValid = this->isValidIndex(nRow, nCol);
                    
                    if (indexValid && !(nRow == row && nCol == col))
                    {
                        int neighborIndex = this->indexForRowColumn(nRow, nCol);
                        Creature* neighbor = gridArray.at(neighborIndex);
                        
                        if (neighbor->getIsAlive())
                        {
                            int livingNeighbors = currentCreature->getLivingNeighborsCount();
                            currentCreature->setLivingNeighborsCount(livingNeighbors + 1);
                        }
                    }
                }
            }
        }
    }

Now it's your turn to write `updateCreatures()`. Create it in *Grid.cpp*. You will need to create a double-nested for-loop like we did in `countNeighbors()` to access every creature in the grid. Look over the code in `countNeighbors()` if you need a refresher on
how to do that.

Next you need to create an if / else if statement. In the if statement,
check if the Creature's `livingNeighbors` property is set to 3. If it is,
that means it has 3 live neighbors so you want to set its `isAlive`
property to `true`. In the else if you want to check if the Creature has
less than or equal to 1 living neighbors or more than or equal to 4. If
either are true, set the creature's `isAlive` property to `false`.

Once you've completed `updateCreatures()`, run your game and see what
happens. If you populate some cells and then tap start, you should see
the game run properly. Try some of [these popular
patterns](http://en.wikipedia.org/wiki/Conway%27s_Game_of_Life#Examples_of_patterns)
and see if they behave as expected.

The only thing that should be missing is the count of live creatures. To
make the label update properly, reset the `populationCount` property at the
beginning of your `updateCreatures()` method by setting it to 0. Create an if statement at the end of the for loop that checks if the creature is alive, if they are you need to increment `populationCount` by 1.

Run the game again - you should be done!

Just in case you get stuck
==========================

You can find the solution for this tutorial on
[GitHub.](https://github.com/MakeSchool/GameOfLife-Cpp)