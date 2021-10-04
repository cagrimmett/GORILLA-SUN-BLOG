---
title: Making of&#58; Grand Canyon
author: Ahmad Moussa
categories:
  - p5js
description: An aesthetically minimalistic sketch that is based on a boolean grid and perlin noise
thumbnail_path: https://gorillasun.de/assets/images/2021-08-25-Making-of-Gateway/gateway.gif
published: true
---

<span class="image fit"><img src="https://gorillasun.de/assets/images/2021-08-25-Making-of-Parasite/parasite.gif" alt="" /></span>

<h2>Hacking a Beesandbombs Sketch</h2>
We'll be using p5js to recreate the sketch, and put our own spin on it. We'll break the sketch up into digestible portions and work from there! Quick index goes here:

1. <a href='#diag'>A Boolean Grid</a>
2. <a href='#wiggle'>Adjustable canvas size</a>
3. <a href='#size'>Drawing the Grid</a>
4. <a href='#atten'>Attenuating the Movement</a>
5. <a href='#fade'>Fading Strokes in and out</a>
6. <a href='#offset'>Offsetting every other loop</a>
7. <a href='#aesthetic'>Aesthetic Touches</a>


<h2>A Boolean Grid</h2>
We'll start off with creating a boolean grid. This won't make much sense at this point, but it's essentially what the entire sketch builds upon. In a nutshell, the grid will determine the positions into which we're allowed to draw:

<pre><code>function setup() {
  w = min(windowWidth, windowHeight)
  createCanvas(w, w);

  padding = 30
  spacing = 5

  bools = []
  for(x = padding; x<w-padding; x+=spacing){
    row = []
    for(y = padding; y<w-padding; y+=spacing){
      row.push(0)
    }
    bools.push(row)
  }
}
</code></pre>

At this point I've covered grids countless times in my blog posts, and yet again we'll make use of one! Here we're creating a 2D array that we fill with zeroes. It's size in width and height are determined by the parameters that we specify beforehand: padding and spacing. I used to call these 'off' and 'spc' previously, but for sake of clarity these make a lot more sense. Padding is essentially the horizontal and vertical distance between the border of the canvas and the grid itself. Spacing is simply the distance between points in the grid.

<h2>Adjustable canvas size</h2>
Before we go any further into the details of this sketch, I'd like to make an adjustment to the manner in which we create the grid. What if we would like to have a grid that is not square in shape? For instance we would like to have a wide aspect ratio or have a canvas with a portrait aspect ratio?

We could do something like this:
<pre><code>w = min(windowWidth, windowHeight)
wx = w
wy = w
createCanvas(wx, wy);
</code></pre>

In this manner we can now simply add a scaling factor to either the width of the height of the canvas to get the desired shape. So for example if we want a 16:9 aspect ratio:

<pre><code>w = min(windowWidth, windowHeight)
wx = w*1.6
wy = w*0.9
createCanvas(wx, wy);
</code></pre>

Neat right? I've been using this trick to have differently size images when posting to Twitter! And naturally we'll need to account for the different width and height of the canvas and modify the loop that creates the boolean grid:

<pre><code>bools = []
for(x = padding; x<wx-padding; x+=spacing){
  row = []
  for(y = padding; y<wy-padding; y+=spacing){
    row.push(0)
  }
  bools.push(row)
}
</code></pre>

Our code would now look as follows:

<script src="//toolness.github.io/p5.js-widget/p5-widget.js"></script>
<script type="text/p5" data-p5-version="1.2.0" data-autoplay data-preview-width="350" data-height="400">
function setup() {
  w = min(windowWidth, windowHeight)
  wx = w*1.6
  wy = w*0.9
  createCanvas(wx, wy);

  padding = 30
  spacing = 5

  bools = []
  for(x = padding; x<wx-padding; x+=spacing){
    row = []
    for(y = padding; y<wy-padding; y+=spacing){
      row.push(0)
    }
    bools.push(row)
  }
}
</script>
<p></p>

This doesn't draw anything yet but bear with me!

<h2>Drawing the Grid</h2>
Now we've got the boolean grid, but we're not storing the location of the dots anywhere, we can however make use of the padding and spacing parameters as well as the dimensions of the grid to determine the positions:

<pre><code>function draw(){
  for(i = 0; i<bools.length; i++){
    for(j = 0; j<bools[0].length; j++){
      point(i * spacing + padding, j * spacing + padding)
    }
  }
}
</code></pre>

Running this code will draw a grid of points to the canvas now:

<script src="//toolness.github.io/p5.js-widget/p5-widget.js"></script>
<script type="text/p5" data-p5-version="1.2.0" data-autoplay data-preview-width="350" data-height="400">
function setup() {
  w = min(windowWidth, windowHeight)
  wx = w*1
  wy = w*1
  createCanvas(wx, wy);

  padding = 30
  spacing = 5

  bools = []
  for(x = padding; x<wx-padding; x+=spacing){
    row = []
    for(y = padding; y<wy-padding; y+=spacing){
      row.push(0)
    }
    bools.push(row)
  }
}

function draw(){
  for(i = 0; i<bools.length; i++){
    for(j = 0; j<bools[0].length; j++){
      point(i * spacing + padding, j * spacing + padding)
    }
  }
}
</script>
<p></p>

<h2>Drawing a connected path through this grid</h2>
The next thing we'd like to do, is drawing a connected path horizontally through this grid. We can do this by setting a random entry in each column to be true and then connecting these entries with lines. First things first, how do we set a random value in each column to true? We could do this during setup:

<pre><code>bools = []
for(x = padding; x<wx-padding; x+=spacing){
  row = []
  for(y = padding; y<wy-padding; y+=spacing){
    row.push(0)
  }
  randomRowIndex = int(random(row.length))
  row[randomRowIndex] = 1
  bools.push(row)
}
</code></pre>

Making sure that we did this correctly, we'll draw an ellipse at these grid locations:

<script src="//toolness.github.io/p5.js-widget/p5-widget.js"></script>
<script type="text/p5" data-p5-version="1.2.0" data-autoplay data-preview-width="350" data-height="400">
function setup() {
  w = min(windowWidth, windowHeight);
  wx = w * 1;
  wy = w * 1;
  createCanvas(wx, wy);

  padding = 30;
  spacing = 10;

  bools = [];
  for (x = padding; x < wx - padding; x += spacing) {
    row = [];
    for (y = padding; y < wy - padding; y += spacing) {
      row.push(0);
    }
    randomRowIndex = int(random(row.length));
    row[randomRowIndex] = 1;
    bools.push(row);
  }
}

function draw() {
  for (i = 0; i < bools.length; i++) {
    for (j = 0; j < bools[0].length; j++) {
      if(bools[i][j]){
        ellipse(i * spacing + padding, j * spacing + padding,spacing/2);
      }else{
        point(i * spacing + padding, j * spacing + padding);
      }
    }
  }
}
</script>
<p></p>

Looks good! Next we'll want to connect these grid locations with a line:

<script src="//toolness.github.io/p5.js-widget/p5-widget.js"></script>
<script type="text/p5" data-p5-version="1.2.0" data-autoplay data-preview-width="350" data-height="400">
function setup() {
  w = min(windowWidth, windowHeight);
  wx = w * 1;
  wy = w * 1;
  createCanvas(wx, wy);

  padding = 30;
  spacing = 10;

  bools = [];
  for (x = padding; x < wx - padding; x += spacing) {
    row = [];
    for (y = padding; y < wy - padding; y += spacing) {
      row.push(0);
    }
    randomRowIndex = int(random(row.length));
    row[randomRowIndex] = 1;
    bools.push(row);
  }

  strokeWeight(2)
}

prevI = 0
prevJ = 0
function draw() {
  for (i = 0; i < bools.length; i++) {
    for (j = 0; j < bools[0].length; j++) {
      if(bools[i][j]){
        if(i>0){
          line((i-1) * spacing + padding,
               j * spacing + padding,
              i * spacing + padding,
               j * spacing + padding)

          line((i-1) * spacing + padding,
               j * spacing + padding,
              prevI * spacing + padding,
               prevJ * spacing + padding)
        }
        prevI = i
        prevJ = j
      }else{
        point(i * spacing + padding, j * spacing + padding);
      }
    }
  }
  noLoop();
}
</script>
<p></p>

Now you see, this doesn't look that nice to be honest. The peaks and troughs are all over the place. It would look much nicer if this path was a little smoother and controlled. We could do this by replacing the random selection of the row index with perlin noise:

<pre><code>
rez = 0.01
randomRowIndex = int(noise(x*rez,y*rez)*row.length);
</code></pre>

If you're not familiar with the magical p5 noise() function have a look at <a href='https://gorillasun.de/blog/Introduction-to-Perlin-Noise-in-P5JS-and-Processing'>this tutorial</a> I've written.

The rez variable here stands for resolution, and specifies how fine we want this perlin noise to be. The lower the rez arameter, the smoother the path. We'll give as input the x and y location of the grid point, but you can alternatively feed in other variables. We'd get a result that looks as follows:

<script src="//toolness.github.io/p5.js-widget/p5-widget.js"></script>
<script type="text/p5" data-p5-version="1.2.0" data-autoplay data-preview-width="350" data-height="400">
function setup() {
  w = min(windowWidth, windowHeight);
  wx = w * 1;
  wy = w * 1;
  createCanvas(wx, wy);

  padding = 30;
  spacing = 5;

  bools = [];
  for (x = padding; x < wx - padding; x += spacing) {
    row = [];
    for (y = padding; y < wy - padding; y += spacing) {
      row.push(0);
    }
    randomRowIndex = int(noise(x*0.01,y*0.01)*row.length);
    row[randomRowIndex] = 1;
    bools.push(row);
  }

  strokeWeight(1)
}

prevI = 0
prevJ = 0
function draw() {
  for (i = 0; i < bools.length; i++) {
    for (j = 0; j < bools[0].length; j++) {
      if(bools[i][j]){
        if(i>0){
          strokeWeight(2)
          line((i-1) * spacing + padding,
               j * spacing + padding,
              i * spacing + padding,
               j * spacing + padding)

          line((i-1) * spacing + padding,
               j * spacing + padding,
              prevI * spacing + padding,
               prevJ * spacing + padding)
        }
        prevI = i
        prevJ = j
      }else{
        strokeWeight(1)
        point(i * spacing + padding, j * spacing + padding);
      }
    }
  }
  noLoop();
}
</script>
<p></p>

This looks much better. Also notice that I adjusted the strokeWeight of the line and the points as well as the spacing in between them. This way things look much neater, to me at least. Feel free to make some changes and @ me on Twitter!

<h2>Adding Details</h2>

This is actually already the bulk of this sketch, but we can add a couple of ornaments to make things a little bit more interesting. For example adding a little rectangle here and there that looks like a boulder:

<pre><code>
if(random()>0.9){
  rect( (i-1) * spacing + padding,
        j * spacing + padding, spacing)
}
</code></pre>

And some rain drops coming from the sky above the mountains. For this we'll need to check if we already drew the horizontal line, if we did we toggle the boolean variable 'below' to stop drawing rain drops:
<pre><code>
if(random()>0.9 && !below && j>0){
  line(i * spacing + padding, j * spacing + padding,
      i * spacing + padding, (j-1) * spacing + padding)
}
</code></pre>


<script src="//toolness.github.io/p5.js-widget/p5-widget.js"></script>
<script type="text/p5" data-p5-version="1.2.0" data-autoplay data-preview-width="350" data-height="400">
function setup() {
  w = min(windowWidth, windowHeight);
  wx = w * 1;
  wy = w * 1;
  createCanvas(wx, wy);

  padding = 30;
  spacing = 5;

  bools = [];
  for (x = padding; x < wx - padding; x += spacing) {
    row = [];
    for (y = padding; y < wy - padding; y += spacing) {
      row.push(0);
    }
    randomRowIndex = int(noise(x*0.01,y*0.01)*row.length);
    row[randomRowIndex] = 1;
    bools.push(row);
  }

  strokeWeight(1)
}

prevI = 0
prevJ = 0
function draw() {
  for (i = 0; i < bools.length; i++) {
    below = false
    for (j = 0; j < bools[0].length; j++) {

      if(bools[i][j]){
        below=true
        if(i>0){
          strokeWeight(2)
          line((i-1) * spacing + padding,
               j * spacing + padding,
              i * spacing + padding,
               j * spacing + padding)

          line((i-1) * spacing + padding,
               j * spacing + padding,
              prevI * spacing + padding,
               prevJ * spacing + padding)
        }
        prevI = i
        prevJ = j

        if(random()>0.9){
          rect( (i-1) * spacing + padding,
               j * spacing + padding, spacing)
        }
      }else{
        strokeWeight(1)
        point(i * spacing + padding, j * spacing + padding);

        if(random()>0.9 && !below && j>0){
          line(i * spacing + padding, j * spacing + padding,
              i * spacing + padding, (j-1) * spacing + padding)
        }
      }
    }
  }

  noLoop();
}
</script>
<p></p>