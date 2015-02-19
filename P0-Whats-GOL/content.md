---
title: What's the Game of Life?
slug: what-game-of-life
---

In this tutorial you are going to familiarize yourself with Objective-C
and a game development tool called SpriteBuilder by making a mobile
version of Conway's Game of Life. If you've never heard of Conway's Game
of Life,
[Wikipedia](http://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) has a
great article. Nearly every programmer has written a version of this
game at some point in their careers and wasted lots of time staring at
cool shapes morphing. Consider this your initiation :p


TL;DR
=====

There is a grid of cells. A cell is either alive or dead. If a cell has
less than two live neighbors, it dies. If it has more than three
neighbors, it dies. If a live cell has exactly two or three neighbors,
it stays alive and if a dead cell has exactly three neighbors, it comes
to life.

Try placing a few live cells and then hitting the next button to run one
round. The Wikipedia article has some great examples of common patterns
that produce cool effects. Press the animate button to continuously run
the game. Play around with it a little and come back when you're ready.
We'll wait :)

<!--
 Original source code written by Ron de Jong, 20 Oct. 2011 at http://www.codeproject.com/Articles/271154/HTML5-Game-of-Multi-Life.
 Accessed September 4, 2012.
 Licensed under The Code Project Open License (CPOL): http://www.codeproject.com/info/cpol10.aspx.
 Latest version at date of access was CPOL version 1.02.

 Source code modified by Brian Chu for MakeGamesWithUs Inc.
 Canvas size made smaller
 Dropdown menu removed. Cells can only be magenta colored.
 -->

<style type="text/css">
select
{
    font-size: 10pt;
}
div#params
{
    font-size: 10pt;
    font-family: verdana, arial, sans-serif;
    margin: 10px;
}
canvas
{
    border-color: Gray;
    border-width: thin;
    position: absolute;
    top: 0px;
    left: 0px;
}
#canvas2
{
    background-color: #f5f5f5;
}
button
{
    width: 80px;
    color: #393939;
}

</style>

<script type="text/javascript" >
    function LifeTorus(size) {
        this.size = size;
        var count = size * size;
        this.torus = new Array(count);

        this.clear = function () {
            for (var i = 0; i < count; i++)
                this.torus[i] = 0;// 0 means empty for convenience and speed
        };

        // returns count of the number of neighbours of each kind
        this.getNeighbours = function (x, y) {
            var count = [0, 0, 0, 0, 0];
            // prev row
            count[this.get(x - 1, y - 1)]++;
            count[this.get(x, y - 1)]++;
            count[this.get(x + 1, y - 1)]++;

            // this row
            count[this.get(x - 1, y)]++;
            count[this.get(x + 1, y)]++;

            // next row
            count[this.get(x - 1, y + 1)]++;
            count[this.get(x, y + 1)]++;
            count[this.get(x + 1, y + 1)]++;

            return count;
        };

        this.get = function (x, y) {
            return this.torus[this.getIndex(x, y)];
        };

        this.set = function (x, y, value) {
            this.torus[this.getIndex(x, y)] = value;
        };

        // Treats the two dimensional arreay as a torus, i.e.
        // the top and bottom edges of the array are adjacent and the left and right edges
        // are adjacent.
        this.getIndex = function (x, y) {
            if (x < -1 || y < -1 || x > size || y > size)
                throw "Index out of bounds";
            if (x == -1)
                x = size - 1;
            else if (x == size)
                x = 0;
            if (y == -1)
                y = size - 1;
            else if (y == size)
                y = 0;
            return x + y * this.size;
        };

        this.clear();
    }

    function relMouseCoords(event) {
        var totalOffsetX = 0;
        var totalOffsetY = 0;
        var canvasX = 0;
        var canvasY = 0;
        var currentElement = this;

        do {
            totalOffsetX += currentElement.offsetLeft;
            totalOffsetY += currentElement.offsetTop;
        }
        while (currentElement = currentElement.offsetParent)

        canvasX = event.pageX - totalOffsetX;
        canvasY = event.pageY - totalOffsetY;

        return { x: canvasX, y: canvasY }
    }
    HTMLCanvasElement.prototype.relMouseCoords = relMouseCoords;
</script>

<div id='params'>
<button onclick="clearGame()">Clear</button>
<button onclick="advance()" >Next</button>
<button id="btnAnimate" onclick="animate()">Animate</button>
<span id="generation" style="width: 130">Generation: 0</span>
<span id="population" style="width: 130">Population: 0</span>
</div>

<div style="position:relative; height: 401px;">
<canvas id='canvas2' width='401' height='401' on></canvas> <!-- Lowest in Z-order - provides background -->
<canvas id='canvas1' width='401' height='401' on>Canvas is not supported by this browser.</canvas>
</div>

<script type="text/javascript" >

// Keep a torus for the current and next generation
var _size = 40;
var _cellSize = 10;
var _torus1 = new LifeTorus(_size);
var _torus2 = new LifeTorus(_size);
var _animate = false;
var _generation = 0;
var isMouseDown = false;

function clearGame() {
    _torus1.clear();
    _generation = 0;
    generation.textContent = "Generation: 0";
    render();
    updatePopulation();
}

function animate() {
    _animate = !_animate;
    if (_animate) {
        advance();
        btnAnimate.textContent = "Stop";
    } else {
        btnAnimate.textContent = "Animate";
    }
}

function advance() {
    // torus1 contains the current model, process into torus2 then swap the
    // references so torus1 refers to the next generation
    var _population = 0;
    for (var x = 0; x < _size; x++)
        for (var y = 0; y < _size; y++) {
            var neighbours = _torus1.getNeighbours(x, y);// dim 5 array
            var alive = 0;
            var kind = _torus1.get(x, y);
            if (kind > 0) {
                // its alive - it will stay alive if it has 2 or 3 neighbours
                var count = neighbours[kind];
                alive = (count == 2 || count == 3) ? kind : 0;
            }
            else {
                // Its dead but will be born if any "kind" has exactly 3 neighbours
                // This isn't "fair" but we use the first kind that has three neightbours
                for (kind = 1; kind <= 4 && alive == 0; kind++) {
                    if (neighbours[kind] == 3)
                        alive = kind;
                }
            }
            _torus2.set(x, y, alive);
            if (alive)
                _population++;
        }

    var temp = _torus1; // arrays are only references!
    _torus1 = _torus2;
    _torus2 = temp;
    render();
    generation.textContent = "Generation: " + String(++_generation);
    population.textContent = "Population: " + String(_population);
    if (_animate)
        setTimeout("advance()", 50);
}

function renderCanvas(canvas, size, torus) {
    // read from LifeTorus and write to canvas
    var context = canvas.getContext('2d');
    context.fillStyle = '#ff7f50';
    context.clearRect(0, 0, size * _cellSize, size * _cellSize);
    for (var x = 0; x < size; x++)
        for (var y = 0; y < size; y++) {
            var kind = _torus1.get(x, y) - 1;
            if (kind >= 0) {
                context.fillStyle = '#ff1493';
                context.fillRect(x * _cellSize, y * _cellSize, _cellSize, _cellSize);
            }
        }
}

function render() {
    renderCanvas(canvas1, _size, _torus1);
}

function drawGrid() {
    // Only ever called once!
    var context = canvas2.getContext('2d'); // canvas2 is the background canvas
    context.strokeStyle = '#808080';
    context.beginPath();
    for (var i = 0; i <= _size; i++) {
        // Draw vertical lines
        context.moveTo(i * _cellSize + 0.5, 0.5);
        context.lineTo(i * _cellSize + 0.5, _size * _cellSize);
        // Draw horizontal lines
        context.moveTo(0.5, i * _cellSize + 0.5);
        context.lineTo(_size * _cellSize, i * _cellSize + 0.5);
    }
    context.stroke();
}

drawGrid();

canvas1.onmousedown = function canvasMouseDown(ev) {
    isMouseDown = true;
    var x = ev.pageX - this.offsetLeft;
    var y = ev.pageY - this.offsetTop;
    var coords = this.relMouseCoords(ev);
    setPoint(coords.x, coords.y);
}

canvas1.onmouseup = function canvasMouseDown(ev) {
    isMouseDown = false;
}

canvas1.onmousemove = function canvasMouseDown(ev) {
    if (isMouseDown) {
        var coords = this.relMouseCoords(ev);
        setPoint(coords.x, coords.y);
    }
}

function setPoint(x, y) {
    // convert to torus coords
    var i = Math.floor(x / _cellSize);
    var j = Math.floor(y / _cellSize);

    // Which kind
    var kind = 1;

    _torus1.set(i, j, kind);
    render();
    updatePopulation();
}

function updatePopulation() {
    var _population = 0;
    for (var x = 0; x < _size; x++)
        for (var y = 0; y < _size; y++) {
            if (_torus1.get(x, y))
                _population++;
        }

    population.textContent = "Population: " + String(_population);
}
</script>