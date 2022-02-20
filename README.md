# ai-with-brainjs
IBM Developer Technology Sandbox preview: Helping developers explore new technologies Learn more
Skip to main content
IBM Developer
Topics
Products & Services
Community
Open source at IBM
Search



Front-end development
Articles
Code Patterns
Podcasts
Open Projects
Tutorials
Videos
Community
Blog Posts
Related topics
Mobile
Web development
JavaScript
Tutorial

Build and train a neural network with nothing but JavaScript using Brain.js
Learn the basics and best practices of machine learning along the way
 Save
 Like
By Jeff Mahoney 
Published July 27, 2020

Introduction

AI and machine learning is an exciting new frontier for software development. And while new tools, sites, and other resources are constantly emerging, most of them are written in Python (or perhaps R). If you are a front-end developer (whose favorite language is likely JavaScript), you might be left wondering if it is possible for you to explore this new world.

Fortunately, the answer to that question is “yes.” There are libraries out there that allow you to practice machine learning without knowing a line of Python. In this tutorial, we will explore one of them — Brain.js — and show how it’s possible to build, train, and use a deep neural network (DNN) with nothing but JavaScript.

Learning objectives

We will step through a pre-built DNN. We will show how it works from the UI, delve into the code that connects the UI to the DNN, then explain the code that actually creates and trains the DNN. We will then pull back and explain some of the base concepts of what DNNs are, how they work, and how people create and improve them. Along the way, we will discover how DNNs allow us to achieve impressive results with surprisingly little code.

Prerequisites

This tutorial assumes no prior experience with DNNs or machine learning.

For the tutorial, you will need:

Basic familiarity with JavaScript
Basic familiarity with Git and GitHub
Estimated time

Completing this tutorial should take about 30-60 minutes.

Steps

Step 1. Draw and predict

After you’ve cloned the repo, open the number-evaluator.html file in the browser of your choice. You should see the following when it loads.

Figure 1

This is the UI for a number prediction application that uses a DNN to guess what you have drawn in its grid.

Try drawing a number in the grid; it can be any value from 0 to 9 (one digit only). You draw numbers by clicking on a square in the grid, holding down your left mouse key, and dragging it around, releasing when done.

After you’ve drawn something, click the Get Assessment button. Doing so should produce feedback that looks like Figure 2.

Figure 2

To make another prediction, click the Clear Input button and draw another number in the grid, then click Get Assessment again.

Here’s what just happened

Hopefully by now, you’re intrigued as to how this program works. What has happened so far is that you have interacted with a pre-trained DNN.

On a very basic level, you are creating input — the number you draw — and feeding it to a DNN. The DNN, in turn, is using what it has been previously taught (more on that later) to make an educated guess or a prediction. The code displays this prediction in the UI, with some added detail, like yellow highlighting to make the network’s choice more obvious.

Step 2. Implement a DNN in JavaScript with the Brain.js library

We are now going to walk through the code so you can see what implementing a DNN in JavaScript with the Brain.js library looks like. Once we’ve done that, we will step back and talk more generally about how DNNs work and refer back to the code we’ve seen to tie it all together.

When we click the Get Assessment button, we can see that it invokes a function called getAssessment():

<button onclick="getAssessment()" style="width:49%;">
    Get Assessment
</button>

Show more
When we scroll farther down in the number-evaluator.html file, we can see the definition of getAssessment():

function getAssessment() {
    const inputArray = captureAndTransformInput()
    let ourNetwork = ourNeuralNetwork
    const result = arrayToHTML(resultToArray(ourNetwork.run(inputArray)))

    document.getElementById('assessment').innerHTML = result
}

Show more
The getAssessment() function is something we created so our UI can talk to the DNN object created by the Brain.js library (more on that in a bit).

The first thing we need to do, if we want to use the DNN, is acquire data about what we drew on the screen. We need to give our DNN something to examine. We do this with another function we’ve created for our demo called captureAndTransformInput(). This function iterates over the grid on our web page and examines which boxes got colored red and which did not. It then takes all of them and transforms the whole into an array of 1’s and 0’s, and returns that as the result:

function captureAndTransformInput() {
    const boxes = Array.from(document.getElementsByClassName('cell'))
    const toOnesOrZeros = box => box.style.backgroundColor === 'red' ? 1 : 0
    return boxes.map(toOnesOrZeros)
}

Show more
We need to perform this transformation because the DNN provided to us by the Brain.js library will not know what to do with the raw data (the color info from the HTML page). Our DNN only knows how to process a list of 1’s and 0’s.

We are, to use deep learning parlance, creating an input vector for our DNN with the captureAndTransformInput() function.

Step 3. Use DNN to classify what we drew and determine its accuracy

All DNNs require an input vector to do their work. Even neural-network examples you might have heard of, such as those that process photographs to determine if they contain cats or dogs, don’t work with photos directly. Instead, a script (usually Python) takes the pixel information from the photo and unspools it into a long list of values, which are usually numbers representing the different pieces of each RGB value for each pixel in the photo. The result is called an input vector, and for a photo, it can contain many values. We are doing a similar — although simpler — operation with captureAndTransformInput().

Now that we have our input vector, we can use our DNN to classify what we have just drawn and determine how accurate it is.

When you use the Brain.js library, you create an object that holds information about your DNN. One of facets of this object is a method called run(). This is what actually generates the prediction that ultimately winds up on the web page we see:

ourNeuralNetwork.run(...)

Show more
But before we can actually use our DNN object, we have to train it and set some parameters first. We will explain that shortly, but for now we will continue seeing how the network’s prediction makes it back to the web page.

The Brain.js run() method takes an input vector as an argument and returns a result. This result is a JSON payload that contains information about our DNN’s educated guess about what number we drew. For our tutorial, we feed the run() method the output of our captureAndTransformInput() function:

const inputArray = captureAndTransformInput()
const result = arrayToHTML(resultToArray(ourNeuralNetwork.run(inputArray)))
The output of the run() method - i.e., our network’s prediction - is then passed to another “transformer” function we have created called resultToArray():

function resultToArray(resultToConvert) {
    let arrayToReturn = []

    arrayToReturn.push({ label: 'Zero', likelihood: resultToConvert.Zero, topChoice: 0, ordinal: 0 })
    arrayToReturn.push({ label: 'One', likelihood: resultToConvert.One, topChoice: 0, ordinal: 1 })
    arrayToReturn.push({ label: 'Two', likelihood: resultToConvert.Two, topChoice: 0, ordinal: 2 })
    arrayToReturn.push({ label: 'Three', likelihood: resultToConvert.Three, topChoice: 0, ordinal: 3 })
    arrayToReturn.push({ label: 'Four', likelihood: resultToConvert.Four, topChoice: 0, ordinal: 4 })
    arrayToReturn.push({ label: 'Five', likelihood: resultToConvert.Five, topChoice: 0, ordinal: 5 })
    arrayToReturn.push({ label: 'Six', likelihood: resultToConvert.Six, topChoice: 0, ordinal: 6 })
    arrayToReturn.push({ label: 'Seven', likelihood: resultToConvert.Seven, topChoice: 0, ordinal: 7 })
    arrayToReturn.push({ label: 'Eight', likelihood: resultToConvert.Eight, topChoice: 0, ordinal: 8 })
    arrayToReturn.push({ label: 'Nine', likelihood: resultToConvert.Nine, topChoice: 0, ordinal: 9 })

    const byLikelihood = (x, y) => x.likelihood < y.likelihood ? 1 : -1
    const byOrdinal = (x, y) => x.ordinal < y.ordinal ? -1 : 1

    const topChoiceDesignation = (e, i, a) => {
        e.topChoice = i === 0 ? 1 : 0  // mark the first entry (index 0) as the top choice - we can do this because we've already sorted them to ensure that the top choice is the first item 
        return e
    }

    return arrayToReturn.sort(byLikelihood)
        .map(topChoiceDesignation)
        .sort(byOrdinal)
}

Show more
This function takes in prediction data (resultToConvert) and creates a new array (arrayToReturn) that has information we can use to display a result in the UI. The output returned by our DNN will be an object that has 10 properties (Zero, One, Two, Three, Four, Five, Six, Seven, Eight, Nine), each of them with a probability (between 0 and 1) mapped to it.

So, for example, a prediction payload from our DNN (the resultToConvert parameter) might look like this (our DNN is quite confident that whatever it is looking at is the number 4):

// a possible payload for resultToConvert - this data came from the DNN
{
    Zero: 0.1927014673128724,
    One: 1.588071696460247,
    Two: 0.09038684074766934,
    Three: 0.0014367527001013514,
    Four: 94.81642246246338,
    Five: 0.8259298279881477,
    Six: 0.10091685689985752,
    Seven: 0.46726344153285027,
    Eight: 0.30299206264317036,
    Nine: 41.01988673210144
}

Show more
We use this output to build a new array, in which each item has a label property (something we create and assign word values to, like “Zero”, “One,” etc.), a likelihood property (which stores the probability received from the neural network for that item), a topChoice property (which we calculate), and an ordinal property (which we assign values to, like 0 , 1, 2-9). We sort our array by likelihood to determine the DNN’s top pick, then use this information to mark that item’s topChoice property with either a 1 (true) or 0 (false). As a last step, we take our newly constructed array and feed it to a final function — arrayToHTML() — which wraps this information in HTML tags:

function arrayToHTML(arr) {
    let htmlToReturn = ''
    htmlToReturn = arr.map(x => {
        let styleToUse = x.topChoice === 1 ? 'background-color: yellow;' : ''
        return `<div style="${styleToUse}">${x.label} Confidence: ${x.likelihood * 100}%</div>`
    }).join('')

    return htmlToReturn;
}

Show more
This function performs, among other actions, a check on each item it receives to determine if its topChoice property is 1. If it is, it wraps the item in yellow-colored markup. This is the highlighting we see on the web page.

With that, we have seen one complete round trip from the UI to the DNN (as user input) and back again (as DNN output that gets relayed to the UI).

Step 4. Set up Brain.js neural-network object

Earlier, we indicated that the DNN object that the Brain.js library enables us to create requires some setup before we can use it. We will talk more about that here.

In the number-evaluator.html file, we have created a function called activate(). This fires when the HTML document initially loads, and it is within this function where we define, create, and train our DNN.

We do this by invoking the NeuralNetwork() method in the Brain.js library to create a new DNN object for us, like so (brain is a variable available to us from our local copy of the Brain.js library, stored in the local-brain.js file):

// setting up a BrainJS DNN object with two specified hyper parameters
ourNeuralNetwork = new brain.NeuralNetwork({
    activation: 'sigmoid',
    learningRate: 0.1
})

Show more
Hyper parameters

When data scientists and machine-learning engineers create neural networks, they often need to provide a set of initial values for how the network will operate when it is trained. These initial values, or arguments, are known as hyper parameters. They govern how quickly the neural network will adjust how it learns, what kind of operations it will use in its hidden layers, and lots of other facets. In other languages and setups, we often need to supply numerous hyper parameters to train a neural network. With Brain.js, however, we don’t need to supply any at all; the library will provide default values if any hyper parameters are omitted.

In our example above, we are specifying two hyper parameters. The first is a parameter called activation that we have set to sigmoid. An activation function is something a neural network executes in its hidden layers to process inputs from one layer to the next. They are critical for helping a neural network learn because they help control which nodes should keep firing and which should fall silent through each layer. A sigmoid is a kind of activation function. There are many varieties we can use (ReLU, Tanh, among others). Each activation function has its own strengths and weaknesses. Using sigmoid is particularly good for things like categorization, which is what we’re trying to accomplish in this tutorial.

The other hyper parameter we’ve specified is learningRate. The learning rate determines how quickly (or slowly) a neural network will adjust its values when it is trying create a useful prediction model (i.e. when it is trying to learn how to distinguish a 3 from a 7, etc.). Specifying the learning rate, like specifying other hyper parameters, is part art and part science: If you set the rate too low, your network might take an unacceptably long time to train, which can have real financial consequences if you are, for example, renting GPUs on a cloud service; and if you set the rate too high, your network might never converge on what is known as a “global minimum” (a solution). Time and practice will enable you to gain a handle on what good initial values look like.

Step 5. Training our neural network

We’ve been talking about a neural network learning and about how we have to train it, but what does that mean, and what does it look like? In the context of our tutorial app, the training looks like this:

const ourTrainingData = getTrainingData()
ourNeuralNetwork.train(ourTrainingData)

Show more
Basically, we are marshalling some training data (getTrainingData()) and training our DNN with it.

Training is fundamental to how DNNs work. Typically, the way it’s done is with a supervised learning approach: We provide a data set with sample inputs and answers (or labels, also sometimes referred to as ground-truth labels) that describe the correct classification for that data row. When we train a DNN (which Brain.js allows us to start with the helpfully named train() method), and it uses these labels to help it learn.

But how, exactly, does the network learn?

Well, essentially, it starts with random guesses and keeps tweaking its formulations with each pass through the training data we give it until either it runs out of chances (i.e. hits the maximum number of iterations we want it to do, which can be another hyper parameter), or it achieves an accuracy threshhold that we deem acceptable (again, another possible hyper parameter). The number of chances is specified by a hyper parameter called iterations, and its default value is 20000 unless you specify a different one like we did for the learning rate and activation earlier. This means that once you invoke the train() method on the DNN object, the underlying library will iterate over your training data up to 20,000 times in an attempt to learn the mysteries of hand-drawn numbers. The “acceptable accuracy” is set by another hyper parameter called errorThresh. This parameter’s value is default set to 0.005 by the Brain.js library unless you provide it with a different value. Your DNN stops training (learning) once either of these conditions are met.

Each node in the DNN has a set of weights it assigns to every input it receives from all of the nodes in the layer before it. The node then multiplies each input by its associated weight, adds everything together, uses this summed value as an input for whatever activation function we specified (which we talked about earlier), and fires the output of the activation function (a new single value) off to each of the nodes in the next layer. Every node in every hidden layer receives as input the outputs of all of the previous layer’s nodes. All of this computation results in a prediction that the network can then test against the label you gave it to determine how correct or incorrect it was. The network is able to assign a mathematical value to how right or wrong it was for that run, and it compares this value with the previous run’s value to determine if it’s getting better or worse at what it’s supposed to be predicting (in our case, the number we drew). It uses this difference to go back and individually tweak each of its weights in a certain direction (either to increase or decrease them, a process called back propagation), then goes through the whole process again until it either hits the iterations count or falls below whatever value is set for errorThresh. This would be a tedious — and most likely impossible — task for a human to do manually, but something that a computer, aided by ever-more-powerful processing chips, can do handily.

That does sound crazy. How do we do all that?

We don’t actually do any of it – the DNN handles all of it for us. All of this weight-adjusting activity (the training) is invisible to us, although you can track how well your DNN is learning by specifying the log hyper parameter to true in your train() call.

All we need to do is provide the network with some training data (inputs and ground-truth labels), and we can sit back and let the Brain.js DNN object (the ourNeuralNetwork object) do the hard work of building a model. If all goes well, we will create a DNN with a model that can make accurate predictions on number drawings it has never seen before.

Step 6. Train the data

If we peek inside our getTrainingData() function, we can what, exactly, our DNN is using to learn.

Brain.js allows us to specify, through our training data, what our input data will look like and what kind of output we would like to see from the neural network. Its main requirement is that we provide an input and an output property. More information can be found on the Brain.js GitHub repo under Training Options.

Here is one of our training rows:

{ "input": [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 1, 0, 0, 0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 1, 0, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0], 
    "output": { 
        Zero: 1, 
        One: 0, 
        Two: 0, 
        Three: 0, 
        Four: 0, 
        Five: 0, 
        Six: 0, 
        Seven: 0, 
        Eight: 0, 
        Nine: 0 
        } 
    }

Show more
In the input portion of the payload, we are providing a lot of 1’s and 0’s. Recall from earlier that this big list of 1’s and 0’s is what our drawing of a number looks like after it’s been processed by our app’s code. We are telling the Brain.js DNN that this is the “shape” of the input data it should expect (the shape of the input vector) and learn to process.

If we look down further, you can see that we have an output section, which contains 10 properties — one for each of the possible one-digit numbers that the network might provide. But remember that this is training data, so we are indicating, by marking the Zero property to 1 on this particular record, that the big list of 1’s and 0’s in the input property represents a zero to us humans. (Other rows have the 1 value set on other properties; examples for the number 4, for example, would have the Four property set to 1, while Zero would be 0.)

This manually marked output section is the ground-truth label that we were talking about earlier. This is what will help the DNN during its training know how to distinguish a blob of 1’s and 0’s that to us looks like a “7” from one that looks like a “0.” With this output setup, we are telling Brain.js to build us a DNN that returns an object with those 10 properties when it gives us a prediction.

When we run our DNN and give it a drawing of some number, the prediction it returns will be a payload that looks like the output tab above. Again, an example of what the output could look like is this:

{
    Zero: 0.1927014673128724,
    One: 1.588071696460247,
    Two: 0.09038684074766934,
    Three: 0.0014367527001013514,
    Four: 94.81642246246338,
    Five: 0.8259298279881477,
    Six: 0.10091685689985752,
    Seven: 0.46726344153285027,
    Eight: 0.30299206264317036,
    Nine: 41.01988673210144
}

Show more
Above are the properties and the values that our resultToArray() and arrayToHTML() functions we discussed earlier use to do their work.

Note that although each of the values might look like a percentage probability, they do not all have to add up to 1. Each value is simply an individual measure of how strongly the DNN feels that what you drew is what that particular property represents. For the example above, it is most confident that we drew a “4” (since the Four property has the highest value), but it also thinks it’s possible, to a lesser extent, that we might have drawn a “9” (since the value assigned to Nine is the second-largest one).

This is not at all abnormal when dealing with DNNs or other kinds of machine-learning neural networks. As developers, we are used to writing code that returns discrete values like True or False or a string or a number. The world of AI, machine learning, and DNNs, though, is a probabilistic one. So we aren’t looking for a specific return value from the DNN in response to our input. We are instead looking for a relative value in a range of values — in this case, the number property with the highest score the DNN assigned to it.

The model

An important thing to remember about all of this is that the DNN has, through our interaction with it, created a model. A model is essentially an algorithm or giant function that reads input (in our case, what you draw on the screen) and makes a prediction, with varying levels of confidence, about what was fed to it. Creating models is the whole point of building and training DNNs (and other kinds of machine-learning networks). Because once a (useful) model has been created, it can be deployed and used (and improved later, if necessary). As the creator and trainer of the DNN, you won’t usually view the model directly (for a model created from a DNN with millions of nodes, for example, it might be essentially impossible to make heads or tails of). The only thing essential thing you need to know about it is how accurately it performs its task as a predictor or classifier.

If you’ve never studied DNNs or machine learning and this all sounds strange to you, don’t worry – it can take a while to absorb these concepts. In the mean time, you can take pride knowing that you too have joined the ranks of people who have created a machine-learning model!

Not optimized

The neural network we are creating and using in this tutorial is very modest. Most networks used for real-world applications are far, far larger (they could easily have millions of nodes).

In our sample app here, we are creating and training the network each time we load (or reload) the HTML page. Since our network is so small, this is not problematic. It is not, however, good practice. In a real-world system, you would most likely be using a DNN (a model) that had taken days, weeks, or possibly months to train. Training is usually the single most time-consuming part of creating any model/network. You obviously can’t do that each time your app or web page loads — unless your users are amazingly patient. What you would do instead is cache the model (network) and deploy it either to memory for a session or wrap it in API calls that clients could use. While training a DNN is often a time-consuming task, once created, they can often execute their predictions quite quickly.

Since we are creating a new DNN with each time we run the program, though, this gives you some freedom to experiment. You can try adding or removing training data to see how these alterations affect the DNN’s performance. You could try, for example, commenting out half (or all!) of the training examples for the number “5” and see what that did to its performance (particularly if you drew a “5”). You can also look at the Brain.js documentation to see what other hyper parameters you can specify to see how that affects your network’s accuracy.

Supplying your own training data

You might be wondering where all the training data in the getTrainingData() function came from. A companion file called training-data-recorder.html was used to create the entries; see the figure below

Figure 3

The file uses a grid similar to the one in the number-evaluator.html file, except this one is used for creating training samples (i.e. it does not talk to a DNN).

To create a new training example:

Draw a number in the grid.
Important: Categorize what you just drew. You do this by clicking one of the radio buttons beneath the grid. If you are adding a training example for “4,” for example, you would choose the radio button for 4. Selecting a category will enable the Record Input button.
Click the Record Input button. Doing this will create an entry in the text box at the bottom, formatted in such a way as to be useful to the DNN in this tutorial.
Click the Copy Data button to copy whatever is in the text box to your clipboard. Once you’ve done this, you can paste the entry into the getTrainingData() function in the number-evaluator.html file (or you can just manually copy if you prefer). Figure 4

To clear a drawing without removing the training data at the bottom, click Clear Drawing.

To clear the training data at the bottom without removing the drawing, click Clear Training Data.
Summary

In this tutorial, we showed how a DNN could be implemented and trained using nothing but JavaScript, thanks to the Brain.js library — tasks that normally require Python and a library like Keras, Pytorch, or TensorFlow. We demonstrated that an effective number recognizer can be created with surprisingly little code. And we learned some of the basic building blocks and practices of machine learning, like specifying learning rates and activation functions for a DNN (among other hyper parameters). Hopefully, this tutorial has shown you that getting started in AI and machine learning with no prior experience is not only possible but also feasible with a familiar language like JavaScript. Explore the links below to find more resources to expand your AI journey.

