---
layout: subpage
title: Tutorial - Creating a Proficiency Exercise with JSAV
---


This tutorial will walk you through the code needed to create a
simple proficiency exercise.
The user of the exercise is supposed to click array positions in order
from left to right to highlight each cell.
A point is scored for each correctly selected cell.
The options panel lets the user choose from various standard settings
for controlling the exercise, including whether feedback for incorrect
steps should be given, and whether such incorrect steps should be
corrected or not.
For the final code version, see the file [JSAV]/examples/simple-exercise.html.

The [API docs](../exercise/) provide complete documentation for
the various library APIs.

### Setting Up

First, like any HTML file we need a header to define the document
type, title, and the CSS files to be used.

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <title>ShellSort Proficiency Exercise</title>
    <link href="AeBookAV.css" rel="stylesheet" type="text/css" />
    <link rel="stylesheet" href="../css/JSAV.css" type="text/css" />
  </head>
{% endhighlight %}

In this case, we included the standard JSAV.css file, as well as the
CSS file used by OpenDSA textbook components (AeBookAV.css).

Next come the style elements.


{% highlight html %}
<style>
  #jsavcontainer {
    width: 500px;
    height: 150px;
    background-color: #efe;
  }
  p.jsavoutput.jsavline {
    height: 40px;
  }
</style>
{% endhighlight %}

Here we first define the style for ```jsavcontainer```,
which holds the visualization.
We define the dimensions and give it a pale green background color.
```jsavoutput``` is where messages can be written by the
visualization.
In this example it will hold the directions for what to do.
We will define the output field to be of type ```jsavline```,
which is one of the style options provided by the API.
We define its height to be 40 pixels.


Next, we will define the various DOM elements on the HTML page.


{% highlight html %}
<div id="jsavcontainer">
  <a class="jsavsettings" href="#">Settings</a>
  <p align="center" class="jsavexercisecontrols"></p>
  <p class="jsavscore"></p>
  <p class="jsavoutput jsavline"></p>
</div>
{% endhighlight %}

```jsavcontainer``` holds the AV itself.
We have given it two standard elements that typically appear in a
JSAV exercise: the options (or "settings") panel, and the exercise
standard controls.
The settings panel is accessed by clicking on the settings button,
which in this example is in the upper right corner and looks like a
pair of gears.
The standard exercise controls include the ability to regenerate a new
instance of the exercise ("reset"), a slideshow showing the "model
answer", and information about the grade received on the exercise.
We create a placeholder for what will display the current score.
Finally, we position the message output, which is merely a special
type of paragraph.


<p>
Next we load in the various libraries.


{% highlight html %}
<script
   src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js">
</script>
<script src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.8.16/jquery-ui.min.js"></script>
<script src="../lib/jquery.transit.js"></script>
<script src="../build/JSAV.js"></script>
{% endhighlight %}

We use the standard jquery libraries, and we also load the JSAV
library.


Next we make variable definitions specific to the exercise.


{% highlight javascript %}
var arraySize = 8,
    initialArray = [],
    jsavArray,
    av = new JSAV($("#jsavcontainer"));
{% endhighlight %}

```arraySize``` defines the array to be of size 8.
```initialArray``` will hold the array values.
This placeholder is needed since JSAV will have to maintain two
versions of the array object: The one that the user works on, and the
copy displayed when the model answer is presented.
```jsavArray``` will hold a reference to the JSAV array the student is working on.</b>
```av``` is the JSAV visualization object itself.
We pass in ```jsavcontainer``` to bind the container of the
visualization to the DOM element with this ```id```.

Next we deal with giving instructions to the user.

{% highlight javascript %}
av.recorded(); // we are not recording an AV with an algorithm
{% endhighlight %}

By default, when a JSAV visualization is initialized it is expecting a
series of effects to be recorded by running an algorithm.
In this mode, the effects are only recorded and not
visualized/animated.
When making an exercise, we want to animate user operations as soon as
they are performed.
Thus, we call ```av.recorded()``` to end the "recording" mode.
Note that animation state changes are still stored in JSAV's internal
animation state sequence array.


### Write an Initialization Function
We are now ready to specify the pieces to the exercise itself.

The first step in creating an exercise is to write a function that
initializes the exercise.
This method will be called both on startup, and by the "reset"
button.
The ```initialize()``` function should clear the previous
visualization (if there was one) and initialize a new one.
It should also return the structures that will be used in grading to
compare with the model answer.
The name of the function does not have to be "initialize", as
its name will be bound when we create the exercise object,
as shown later in the tutorial.


{% highlight javascript %}
function initialize() {
  if (jsavArray) {
    jsavArray.clear();
    swapIndex.clear();
  }
  av.umsg("Directions: Click on all array elements from left to right to highlight them. Then click on the first and last elements to swap them.");
  initialArray = JSAV.utils.rand.numKeys(10, 100, arraySize);

  jsavArray = av.ds.array(initialArray, {indexed: true});
  swapIndex = av.variable(-1);
  // bind a function to handle all click events on the array
  jsavArray.click(arrayClickHandler);
  return jsavArray;
}
{% endhighlight %}

Our example here is a simplified version of the one used by
the Shellsort Proficiency Exercise. The code start by checking if we have an old instance
of jsavArray, and clears the HTML elements if we do (calls to ```clear()```).
Next, the code generates random numbers with a call to ```JSAV.utils.rand.numKeys```.
The call to ```av.ds.array``` initializes a new JSAV array from
the given array and the given options. The call to ```av.variable``` initializes a new
variable that can be used to store value changes in the animation sequence. See
section "Adding Student Interaction" below for explanation why we need this.
The call to ```av.umsg()``` provides the text of the message
that give directions to the user. Finally, the function
returns the JSAV array.


### Write the Model Answer Function
To be able to grade the student's solution, the exercise will need
to have a model solution.
This will be used both to show the correct solution (as an AV shown to
the student through the "Model Answer" button) and to compare to the
student's answer to judge correctness.
Below is a simple model answer function where the correct solution is
to highlight the array indices from left (index 0) to right
(index arraySize-1) and then swap first and last element.

{% highlight javascript %}
function modelSolution(modeljsav) {
  var modelArray = modeljsav.ds.array(initialArray);
  modeljsav.displayInit();
  for (var i = 0; i < arraySize; i++) {
    modelArray.highlight(i);
    modeljsav.gradeableStep();
  }
  // swap the first and last element
  modelArray.swap(0, arraySize - 1);
  modeljsav.gradeableStep();
  return modelArray;
}
{% endhighlight %}

The function takes a single parameter (```modeljsav``` above) that
is a reference to the JSAV animation of the model answer.
Like the initialization function, the model solution function should
return the structures used in comparing the model solution with
student's solution.


The function above first creates a JSAV array using the data stored
in variable ```initialArray``` in the initialization
function. Next, it marks the current state as the initially displayed state. Without this line,
the first step in the model answer would be empty and the first step would show the array.
It then goes through all indices in turn and highlights the current
index.
The ```gradeableStep()``` function is used to mark states in the model
solution that should be used in grading.
Thus, the model solution can contain an AV with explanations and other
structures, but only the returned structures at steps where ```gradeableStep``` is
called, are graded. Such explanatory steps can be added through call to
```modeljsav.step()```.


### Creating the exercise

The exercise is initialized with a call to ```av.exercise()```.
The function can also take several options, some of which are
required. Given our ```initialize``` and ```modelSolution```
functions, we initialize an exercise as follows.

{% highlight javascript %}
var exercise = av.exercise(modelSolution, initialize,
                    {feedback: "continuous", compare: {class: "jsavhighlight"}});
exercise.reset();
{% endhighlight %}

The parameters that we pass:

 * ```{model: <function>}``` The function to generate the
  model solution. <strong>Required.</strong>
 * ```{reset: <function>}``` The initialization function
  that resets the exercise. <strong>Required.</strong>
 * ```{options: <Object>}``` The only options we specify are ```feedback```
  for setting the feedback mode to be continuous (feedback after each step) and
  ```{compare: <Object or Array>}```. The compare option specifies which
  properties to compare for the structures. In the example above, we
  set the comparison to be the CSS class ```jsavhighlight```,
  since that is used to highlight indices. **Required.**


The full set of options can be found in the
[API documentation](../exercise/).


At this point, the exercise can be graded and model solution shown.
It now remains to write the code that allows the student to construct
her solution.


### Adding Student Interaction

It is simple to attach listeners for user actions (like mouse clicks)
to JSAV data structures. We already had the code
register a click event handler to the array in the ```initialize``` function:

{% highlight javascript %}
// bind a function to handle all click events on the array
jsavArray.click(arrayClickHandler);
{% endhighlight %}

Now let's see how to write the ```arrayClickHandler``` function.
Inside the handler, we need to first decide if we are in "swap mode" or not. This
is done based on whether the last index is highlighted or not.

{% highlight javascript %}
function arrayClickHandler(index) {
  // if last index is highlighted, we are in "swap mode"
  if (this.isHighlight(arraySize - 1)) {
    // when in swap mode, first click on index will store that index
    // and change the font size on the value
    if (swapIndex.value() == -1) {
      swapIndex.value(index);
      // apply the CSS property change to index
      this.css(index, {"font-size": "130%"});
      av.step(); // add a step to the animation
    } else {
      // the second click (swapIndex has some value) will cause
      // the swap of indices index and stored swapIndex
      this.swap(index, swapIndex.value());
      // change the font-size back to normal
      this.css([swapIndex.value(), index], {"font-size": "100%"});
      swapIndex.value(-1);
      exercise.gradeableStep(); // this step will be graded
    }
  } else { // we are in highlight mode
    // highlight the index
    this.highlight(index);
    if (index == (arraySize - 1)) {
      av.umsg("Good, now swap the first and last index");
    }
    // mark this as a gradeable step; also handles continuous feedback
    exercise.gradeableStep();
  }
});
{% endhighlight %}

If we are in  swap mode, we need to check if this is the first or second index for the swap.
On the first click, we need to store that index to be used when second click occurs. Normally
we could store the value in plain JavaScript variable. However, a student can undo a step, which
should also reset the swapIndex value. For this, we have defined swapIndex as a JSAV variable. It has
functions ```value()``` that will return the current value and ```value(newVal)```
that will set the value.

Now, the exercise should be interactive and ready for use!