---
title: Finish the Game of Life!
slug: game-of-life-code
---       

Time to code! In this step we are going to hook up the UI we've created
in SpriteBuilder with the game logic we're going to code in Xcode.

Create a Grid class
===================

First of all let's create the *Grid* class which we just set as a custom
class in SpriteBuilder. Create a new Objective-C class in Xcode and make
it a subclass of *CCSprite*:

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+SpriteBuilder+Tutorial/GOF-GridClass.png)

Create a Creature class
=======================

We will use some object orientation to implement this game - therefore
Creatures will get their own class, too. Repeat the step above and
create a class called *Creature* that also inherits from *CCSprite*.

When you've created the class, open *Creature.h* and and replace the
content of the file with this:

    #import "CCSprite.h"

    @interface Creature : CCSprite

    // stores the current state of the creature
    @property (nonatomic, assign) BOOL isAlive;

    // stores the amount of living neighbors
    @property (nonatomic, assign) NSInteger livingNeighbors;

    - (id)initCreature;

    @end

If the syntax is unfamiliar you may want to keep the Objective-C Primer
open in a separate tab so you can refer to it during this tutorial. This
code will set up two properties which we will be using later to
determine if creatures are alive or not and how many neighbors they
have. With *initCreature* we declare a custom initializer that sets up
the creature with a creature image - we will be implementing this next.

Open *Creature.m* and add this method between <*@implementation>\* and
<*@end*>:

    - (instancetype)initCreature {
      // since we made Creature inherit from CCSprite, 'super' below refers to CCSprite
      self = [super initWithImageNamed:@"GameOfLifeAssets/Assets/bubble.png"];
      
      if (self) {
        self.isAlive = NO;
      }
      
      return self;
    }

This initializer simply sets the image of the creature to *bubble.png*
and initializes the creatures to be not alive.

We are going to implement a second method. Whenever a creature is not
alive, we will make it invisible. The easiest way to connect the
visibility state to the alive state is to create a setter that is called
when the *isAlive* property gets changed.

Add this method after the *initCreature* method and before the <*@end>\*
line:

    - (void)setIsAlive:(BOOL)newState {
      //when you create an @property as we did in the .h, an instance variable with a leading underscore is automatically created for you
      _isAlive = newState;
      
      // 'visible' is a property of any class that inherits from CCNode. CCSprite is a subclass of CCNode, and Creature is a subclass of CCSprite, so Creatures have a visible property
      self.visible = _isAlive;
    }

Now when we set the Creature to be alive it will become visible, when we
set it to be dead it will disappear.

Fill the grid
=============

In this step we are going to implement a method that fills our grid with
Creature instances in *Grid.m*.

Before we get started we should define some variables we will be using.

First declare two properties in *Grid.h* (after <*@interface>\* and
before <*@end*>):

    @property (nonatomic, assign) int totalAlive;
    @property (nonatomic, assign) int generation;

Then, replace the complete content of *Grid.m* with these lines:

    #import "Grid.h"
    #import "Creature.h"

    // these are variables that cannot be changed
    static const int GRID_ROWS = 8;
    static const int GRID_COLUMNS = 10;

    @implementation Grid {
        NSMutableArray *_gridArray;
        float _cellWidth;
        float _cellHeight;
    }

    @end

Here's what these lines do. We import *Creature.h* because we will be
creating creatures. We define two constants that describe the amount of
rows and columns. Additionally we have a couple of private variables.
*\_gridArray* will be a 2d array that will store all the creatures in
our grid. *\_cellWidth* and *\_cellHeight* will be used to place the
creatures on our grid correctly in the method we will be implementing
soon. The other two variables *\_generation* and *\_totalAlive* will be
used to store the current game stats that will be displayed in the game.

Now let's set up the grid! When the Grid class gets loaded a method
called *onEnter* is called. Let's implement this method in *Grid.m* and
call the *setupGrid* method we are going to write next:

    - (void)onEnter
    {
      [super onEnter];
      
      [self setupGrid];
      
      // accept touches on the grid
      self.userInteractionEnabled = YES;
    }

The *onEnter* method is also the right method to activate touch handling
on the grid. Later in this tutorial we will handle incoming touch
events. Now let's implement *setupGrid* method after *onEnter* and
before <*@end*>:

    - (void)setupGrid
    {
      // divide the grid's size by the number of columns/rows to figure out the right width and height of each cell
      _cellWidth = self.contentSize.width / GRID_COLUMNS;
      _cellHeight = self.contentSize.height / GRID_ROWS;
      
      float x = 0;
      float y = 0;
      
      // initialize the array as a blank NSMutableArray
      _gridArray = [NSMutableArray array];
      
      // initialize Creatures
      for (int i = 0; i < GRID_ROWS; i++) {
        // this is how you create two dimensional arrays in Objective-C. You put arrays into arrays.
        _gridArray[i] = [NSMutableArray array];
        x = 0;
        
        for (int j = 0; j < GRID_COLUMNS; j++) {
          Creature *creature = [[Creature alloc] initCreature];
          creature.anchorPoint = ccp(0, 0);
          creature.position = ccp(x, y);
          [self addChild:creature];
          
          // this is shorthand to access an array inside an array
          _gridArray[i][j] = creature;
          
          // make creatures visible to test this method, remove this once we know we have filled the grid properly
          creature.isAlive = YES;
          
          x+=_cellWidth;
        }
        
        y += _cellHeight;
      }
    }

This code requires some explanation. First we calculate the
*\_cellWidth* and *\_cellHeight* by dividing the size of the grid by the
amount of rows and columns. We set up two positioning variables *x* and
*y* and initialize the array that will hold all the Creatures in our
grid. Then we iterate through two nested loops and create an array for
each row of the grid and fill each row with Creatures. After positioning
a Creature we increase the *x* variable, after completing a row we
increase the *y* variable. As a test we set every creature to be alive
so we can be sure this method works.

After you've added this method run the game. It should look like this:

![image](https://s3.amazonaws.com/mgwu-misc/GameOfLife+SpriteBuilder+Tutorial/GOF-GridComplete.png)

Once you have confirmed that everything works out remove these two lines
from the method you just added:

    // make creatures visible to test this method, remove this once we know we have filled the grid properly
    creature.isAlive = YES;

When the player taps a cell, it should access the Creature that lives on
that cell and kill it if it's alive, or bring it to life if its dead!

The good news is that getting touches is very easy in Cocos2D 3.0. Any
class that is a CCNode or inherits from it will automatically call a
method called touchBegan when the player touches it. All we need to do
is create a touchBegan method and it will get called automatically
(because we set *userInteractionEnabled* to *YES* earlier)! Add the
following method to your Grid.m:

    - (void)touchBegan:(CCTouch *)touch withEvent:(CCTouchEvent *)event
    {
        //get the x,y coordinates of the touch
        CGPoint touchLocation = [touch locationInNode:self];
    
        //get the Creature at that location
        Creature *creature = [self creatureForTouchPosition:touchLocation];

        //invert it's state - kill it if it's alive, bring it to life if it's dead.
        creature.isAlive = !creature.isAlive;
    }

Two things to note here. First, we refer to a method called
creatureForTouchPosition which we haven't created yet. We'll do that
next! Second, when you access a property you've created, like isAlive, a
method called setPropertyName is automatically called. In this case,
setIsAlive. If you don't create one yourself, it'll just set the
property to whatever you pass in. In our case, we created a custom one
in Creature.m that not only sets the property, but makes Creatures'
visibility change depending on their state as well! This is why your
Creatures will automatically go from visible to invisible when you tap
on them.

Now let's create creatureForTouchPosition. First, create an empty
method:

    - (Creature *)creatureForTouchPosition:(CGPoint)touchPosition 
    {
       //get the row and column that was touched, return the Creature inside the corresponding cell
    }

Divide the y coordinate of the touch (accessed as touchPosition.y) by
the cellHeight to get the row that was touched. Store that value in an
integer called *row*. Divide the x coordinate of the touch (accessed as
touchPosition.x) by the cellWidth to get the column that was touched.
Store that value in an integer called *column*.

You can now return the creature with the statement *return
\_gridArray[row][column]*.

Run your game and try tapping on the Grid. You should see Creatures
coming to life and dying where you tap.

Set Up Your Main Scene
======================

In *MainScene.m* we will:

-   create a timer that will call an *evolveStep* method on the grid and
    update the stats and gamestate (generation, alive/dead creatures)
-   implement a *play* method that starts the timer
-   implement a *pause* method that stops the timer

Replace the content of *MainScene.m* with these lines:

    #import "MainScene.h"
    #import "Grid.h"

    @implementation MainScene {
        Grid *_grid;
        CCTimer *_timer;
        CCLabelTTF *_generationLabel;
        CCLabelTTF *_populationLabel;
    }

    - (id)init
    {
        self = [super init];
        
        if (self) {
            _timer = [[CCTimer alloc] init];
        }
        
        return self;
    }

    - (void)play
    {
        //this tells the game to call a method called 'step' every half second.
        [self schedule:@selector(step) interval:0.5f];
    }

    - (void)pause
    {
        [self unschedule:@selector(step)];
    }

    // this method will get called every half second when you hit the play button and will stop getting called when you hi the pause button
    - (void)step
    {
        [_grid evolveStep];
        _generationLabel.string = [NSString stringWithFormat:@"%d", _grid.generation];
        _populationLabel.string = [NSString stringWithFormat:@"%d", _grid.totalAlive];
    }

    @end

Since we hooked up the play and pause buttons to the play and pause
methods in SpriteBuilder, the play and pause methods will automatically
get called when we tap them! Note that you cannot run the game before
finishing the next step.

Evolve!
=======

Now for the tricky part. We need to implement the *evolveStep* method in
*Grid.m*.

First, add an *evolveStep* method declaration to *Grid.h*. If you don't
remember how to do that, check the Objective-C primer or another .h in
this project to remind yourself. The return type of evolveStep will be
*void* since it does not return anything. You then need to implement
your evolveStep method in *Grid.m*.

According to the rules of the Game of Life, we need to count how many
live neighbors every cell has every step. If it has 0-1 live neighbors
the Creature on that cell dies or stays dead. If it has 2-3 live
neighbors it stays alive. If it has 4 or more, it stays dead or dies. If
it has exactly 3 neighbors and it is dead, it comes to life!

So we need to go through every Creature, count the number of live
neighbors it has, and update whether it is alive or dead.

Fill your evolveStep method with the following code:

    //update each Creature's neighbor count
    [self countNeighbors];

    //update each Creature's state
    [self updateCreatures];

    //update the generation so the label's text will display the correct generation
    _generation++;

Once we fill out the countNeighbors and updateCreatures methods, we're
done!

Declare countNeighbors in your Grid.h and create it in Grid.m. Fill it
with the following code:

    // iterate through the rows
    // note that NSArray has a method 'count' that will return the number of elements in the array
    for (int i = 0; i < [_gridArray count]; i++) 
    {
      // iterate through all the columns for a given row
      for (int j = 0; j < [_gridArray[i] count]; j++) 
      {
        // access the creature in the cell that corresponds to the current row/column
        Creature *currentCreature = _gridArray[i][j];
        
        // remember that every creature has a 'livingNeighbors' property that we created earlier 
        currentCreature.livingNeighbors = 0;
        
        // now examine every cell around the current one
        
        // go through the row on top of the current cell, the row the cell is in, and the row past the current cell
        for (int x = (i-1); x <= (i+1); x++) 
        {
          // go through the column to the left of the current cell, the column the cell is in, and the column to the right of the current cell
          for (int y = (j-1); y <= (j+1); y++) 
          {
            // check that the cell we're checking isn't off the screen
            BOOL isIndexValid;
            isIndexValid = [self isIndexValidForX:x andY:y];
            
            // skip over all cells that are off screen AND the cell that contains the creature we are currently updating
            if (!((x == i) && (y == j)) && isIndexValid) 
            {
                Creature *neighbor = _gridArray[x][y];
                if (neighbor.isAlive)
                {
                 currentCreature.livingNeighbors += 1;
                }
            }
          }
        }
      }
    }

You'll notice that the above code refers to a method called
*isIndexValidForX: (int) x andY: (int) y*. We are going to give you that
method. Freebie on us! Here it is:

    - (BOOL)isIndexValidForX:(int)x andY:(int)y
    {
      BOOL isIndexValid = YES;
      if(x < 0 || y < 0 || x >= GRID_ROWS || y >= GRID_COLUMNS)
      {
        isIndexValid = NO;
      }
      return isIndexValid;
    }

Now it's your turn to write updateCreatures. Declare the method in
Grid.h, create it in Grid.m. You will need to create a double-nested
for-loop like we did in countNeighbors to access every creature in the
Grid. Look over the code in countNeighbors if you need a refresher on
how to do that.

Next you need to create an if/else if statement. In the if statement,
check if the Creature's livingNeighbors property is set to 3. If it is,
that means it has 3 live neighbors so you want to set its isAlive
property to TRUE. In the else if you want to check if the Creature has
less than or equal to 1 living neighbors or more than or equal to 4. If
either are true, set the Creature's isAlive property to FALSE.

Once you've completed updateCreatures, run your game and see what
happens. If you populate some cells and then tap start, you should see
the game run properly. Try some of [these popular
patterns](http://en.wikipedia.org/wiki/Conway%27s_Game_of_Life#Examples_of_patterns)
and see if they behave as expected.

The only thing that should be missing is the count of live Creatures. To
make the label update properly, add an int called *numAlive* to the
beginning of your updateCreatures method and set it to 0. Inside the
loop that checks if creatures are alive you need to increment numAlive
by 1 for every creature that is alive.

At the very end of your updateCreatures method, set *\_totalAlive =
numAlive;*. Run the game again - you should be done!

Just in case you get stuck
==========================

You can find the solution for this tutorial on
[GitHub.](https://github.com/MakeGamesWithUs/GameOfLife.spritebuilder)