# Four ways of animating an SVG

Let's make the SVG graphic we created during the [last class](https://github.com/zhoyoyo/lede24-svg-with-d3) _move_.

We will do three things with four examples:

1. Animate the SVG graphic without user interaction in two ways:
    - CSS animation
    - Javascript animation
2. Trigger animation with a button

3. Trigger animation with [Scrollama](https://github.com/russellsamora/scrollama)

#### What to do before we start:

- Clone this repo to your computer (not sure how to? check it out [here](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository)) or download the zip file to this repo using the green top button that says `CODE` 
- Navigate to the folder where the files sit on your laptop, and load a local server. Not sure how? Try one of the following: 
    - For all VS Code users, check out this plugin: [How to load changes spontanously on your local server with VS Code](https://www.freecodecamp.org/news/vscode-live-server-auto-refresh-browser/) 
     - If you use Node or would like to learn (because it's so useful), check out [http-server](https://www.npmjs.com/package/http-server)
     - You can use the following line if you use python. Once it says a local server is running, type in `localhost:8080` on your browser. You should see the content of the `index.html` page there!
        ```
        python3 -m http.server --cgi 8080
        ```



## Method 1: CSS Animation 

We can animate the graphic without touching Javascript by adding this to the `custom.css` file: 


```
svg rect {
    animation-name: easeIn;
    animation-duration: 5s;
    animation-iteration-count: infinite;
    animation-timing-function: ease-in-out;
}

@keyframes easeIn {
    from {
        fill-opacity: 0;
    }

    to {
        fill-opacity: 1;
    }
}
``` 

We can animate certain attributes of an SVG graphic with CSS animation, but it doesn't work for data-driven attributes. 

## Method 2: Javascript Animation -- D3 Transition

[d3-traisition](https://github.com/d3/d3-transition) gives us better control of animating from state to state for targeted DOM elements. 

For example, the colors of the squares in our example are connected to the `Happiness` level from the data. If we want to animate their colors specific to the underlying data, we can do that with `d3-transition`. 

Let's say we want all the squares to start from the same grey colors and then transition to data-specific Hapiness levels. We can achieve that by assigning the fill color to the squares by twice, with a [d3-traisition](https://github.com/d3/d3-transition) function in between. 

Add the following Javascript code after you set the x, y, width, and height attributes of the `individualCharts` variable:


```
// start with grey squares, change the "fill" value
individualCharts.style('fill','lightgrey')

const showSquareData = function(){
    
    individualCharts
        .transition().duration(2000)
        .style('fill', d=> {
            if (d.Happiness == '1') {return 'black'}
            else if (d.Happiness == '2') {return 'grey'}
            else {return 'gold'}
        })
        .on('end', hideSquareData)

}

const hideSquareData = function(){

    individualCharts
        .transition()
        .duration(2000)
        .style('fill','lightgrey')
        .on('end', showSquareData)

}

// run the animation.
showSquareData();

```

Don't forget to delete or comment out the CSS animation you created before. Otherwise they might not work in sync with each other. 

##### 💦 Exercise Interlude 1 💦 
*Animate attributes of an SVG that are not accessible in CSS* 

What if we want to animate the width and height of an SVG rectangle? Take 5 minutes to see if you can figure it out yourself. 

##### 💦 Exercise Interlude 2 💦 
*Can we update the shapes to display new data values?* 

There is a column in the dataset (`Happiness_P2`) that shows the happiness level of a second person. How can we update the circle colors to make them represent that new person's happiness levels? 

##### 🔥 Make animation fancier 🔥 

A sequenced animation sets up expectation and is often more ituitive in telling the story. 

_Read: [12 principles of animation](https://www.gamedeveloper.com/blogs/12-principles-for-game-animation)_

How to achieve it: Use the data index to set up a transition `delay`.

## Method 3: Animation triggered by a button click

In the `<body>` of the HTML, let's create a button above our SVG chart:

```
<button id="btn">Animate!</button>
```
Next, we need to bind the function that starts the animation to the button in Javascript. You can achieve this in either D3 or raw Javascript. Add this after your `hideSquareData()`function declaration, and comment out your call `showSquareData()`: 

```
// showSquareData();

// Create the Click event with D3: 
d3.select('button#btn').on('click', showSquareData)
```

We can further split the animations into three steps: 
1. Animate the width and height to show the squares
2. Show the happiness level of the first person
3. Show the hapiness level of the second person

We can achieve this by replacing the current button with three buttons below the button: 

```
<!-- <button id="btn">Animate!</button> -->

<!-- Add these buttons to the page:  -->
<button id="btn1">Show 100 days</button>
<button id="btn2">Person 1 emotions</button>
<button id="btn3">Person 2 emotions</button>

```

And adding the two functions and binding them to the buttons inside Javascript:

```
const showSquares = function() {

    individualCharts
        .transition()
        .duration(2000)
        .attr('width', gridSize)
        .attr('height', gridSize)
}

const showEmotions = function(personID) {
    individualCharts
        .transition()
        .duration(2000)
        .style('fill', d=> {
        if (d[personID] == '1') {return 'black'}
        else if (d[personID] == '2') {return 'lightgrey'}
        else {return 'gold'}
        })
}

d3.select('button#btn1').on('click', showSquares)

d3.select('button#btn2').on('click', function(){
    showEmotions('Happiness')
})

d3.select('button#btn3').on('click', function(){
    showEmotions('Happiness_P2')
})

```

🤔 Graphic changes are subtle to readers. It's common practice to enhance it with a clear text indication. We can do that inside `click` event callback functions by adding a line. 

Add this to the end of the `ShowSquares` function:

```
d3.select("p").html('Here are the 100 days.")

```

Add this to the end of the `showEmotions` function:

```
d3.select("p").html(function(personID){
    if (personID == 'Happiness") {return "This is the emotional ups and downs of Person 1"}
    else {return "This is the emotional ups and downs of Person 2"}
})

```


Clicking a button to trigger an animation may hide critical information from users. That brings us to... 

## Method 4: Animation triggered by page scroll

Scrolling is a powerful way of pacing a sequence of animation. We are using the [sticky-overlay example](https://russellsamora.github.io/scrollama/sticky-overlay/) in [Scrollama](https://github.com/russellsamora/scrollama) for this tutorial.

First, let's set up our DOM elements to match the structure we typically see in a scrollytelling visual: Replace `<div id="my-svg-chart"></div>` with the following:

```
<div id="scroll-content">  <!-- The overall container -->
    <div id="my-svg-chart"></div> <!-- Your chart -->
    <div id="text"> <!-- Your text -->
        <div class="step">1. Make the charts appear</div>
        <div class="step">2. Show Person 1 emotions</div>
        <div class="step">3. Show Person 2 emotions</div>

    </div> 
</div>

```

Next we need to set up the Javascript portion of Scrollama. Let's load the script as how you loaded d3:

```
<!-- add this below your d3 script -->
<script src="https://unpkg.com/scrollama"></script>
```

Then, we copy and paste the Javascript code section from the example and modify it to our use. You can copy and paste this section and put it below the d3 code (inside the `.then()` function) in our script: 

```
// ======================================
// === HERE starts the scrollama code ===
// ======================================

// using d3 for convenience
const scrolly = d3.select("#scroll-content");
const figure = scrolly.select("#my-svg-chart");
const step = scrolly.selectAll(".step"); 

// initialize the scrollama
const scroller = scrollama();

// generic window resize listener event
function handleResize() {

    // 1. update height of step elements
    var stepH = Math.floor(window.innerHeight * 0.75);
    step.style("height", stepH + "px");

    var figureHeight = Math.min(window.innerHeight*0.8,640);
    var figureMarginTop = (window.innerHeight - figureHeight) / 2;

    figure
        .style("height", figureHeight + "px")
        .style("top", figureMarginTop + "px");

    // 3. tell scrollama to update new element dimensions
    scroller.resize();
}

// scrollama event handlers
function handleStepEnter(response) {

    // add color to current step only
    step.classed("is-active", function (d, i) {
        return i === response.index;
    });

    // update graphic based on step
    if (response.index == 0) {
        // 1. Make the charts appear
        showSquares()
    } else if (response.index == 1) {
        // 2. Show Person 1 emotions
        showEmotions('Happiness')
    } else if (response.index == 2) {
        // 3. Show Person 2 emotions
        showEmotions('Happiness_P2')
    }
}

function init() {

    // 1. force a resize on load to ensure proper dimensions are sent to scrollama
    handleResize();

    // 2. bind scrollama event handlers (this can be chained like below)

    scroller
        .setup({
        step: "#scroll-content .step",
        offset: 0.33,
        debug: false
        })
        .onStepEnter(handleStepEnter);

    // 3. watch for window resizing
    window.addEventListener('resize', handleResize);
}

// kick things off
init();

```

Lastly, copy the example's CSS in the style section to `custom.css`:

```

#scroll-content {
    position: relative;
    padding: 1rem;
}

#scroll-content #text {
    position: relative;
    padding: 0;
    max-width: 20rem;
    margin: 0 auto;
}

#scroll-content #my-svg-chart {
    position: -webkit-sticky;
    position: sticky;
    left: 0;
    width: 100%;
    margin: 0;
    -webkit-transform: translate3d(0, 0, 0);
    -moz-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
    /* background-color:#f9f9f9; */
    z-index: 0;
}

#scroll-content .step {
    margin: 0 auto 2rem auto;
    color: #fff;
    background-color: rgba(0,0,0,0.05);
    color: black;
}

```

Now you have a graphic that animates by scrolling the page.

##### 💦 Exercise Interlude 1 💦 
*Make the graphic responsive to window resizing*
Update the x, y, width and height attributes of the squares inside the `handleResize` function.

##### 💦 Exercise Interlude 2 💦 
*Create a photo scroll* 

Use the template in the folder `/scrollama-sticky-example` to create a photo gallery. Remember to change three places:
- HTML: add photos to your `figure` container
- CSS: make sure the visual is positioned correctly
- Javascript: Add hooks to make the visual appear inside `handleStepEnter()`