# Getting Started

Dexter is a platform that makes it easy for developers to connect third-party APIs and build apps that automate your digital life. Everyday we use a handful of tools that do one thing and do it well, whether that’s ordering lunch or messaging the office a meme. However, more complex tasks require a combination of different solutions. Dexter is here to simplify connecting these services. With Dexter, you can connect the web the way you want.

## Hello World

To get started you need a Dexter account. Dexter is currently in private beta so you'll need an invite code. If you don't have one you can request one at [rundexter.com](https://rundexter.com/#section-6). Once you have an account, sign up and take the walkthrough to build your first app. 

## Detailed Walkthrough

The walkthrough gives you the basic instructions as you click through, but here's some more info in case you get stuck, confused, or just want to know more: 

1. **Create a New App** - Dexter is both for builders and consumers. We start with the building. Click the new app button on the top right of your screen. <img src="/images/screenshots/01-rounded.png" class="image-shadow">  

1. **Name Your App** - The tutorial autopopulates the title and description field with the details of the app we're going to build: a <a href="http://www.google.com/search?q=define:bookmarklet" target="_blank">bookmarklet</a> that sends the url of the website on your desktop screen to your mobile phone via text message so that you can take it with you on the road! <img src="/images/screenshots/02-rounded.png" class="image-shadow">  

1. **Place Your First Trigger** - A trigger step is simply a way for the outside world to communicate with a Dexter app and kickoff some data processing. In this tutorial the trigger is the bookmarklet since it's going to let us know when you want your text message. The way we notify the workflow about a trigger is to drag it onto the light gray bar at the top of the canvas. That's our trigger bar! <img src="/images/screenshots/03-rounded.png" class="image-shadow">  

1. **Place Your First Module** - A module does the heavy lifting. It takes the data from the previous step(s) -- in this case the bookmarklet -- and acts on it. For our first app we want to send ourselves a text message. How is that done? Well by dragging the Dexter SMS module onto the canvas and dragging the orange outlet on the bookmarklet to the input of the Dexter SMS step. <img src="/images/screenshots/04-rounded.png" class="image-shadow">  


1. **Connect The Data** - The last step is telling Dexter how the data moves in between steps. To do this, we click the configure button and bring up the configure panel. Does this panel remind you of the back of your VCR? Good, that's what we meant to do. Each module has outputs and inputs. In this case the bookmarklet module outputs the URL of the page you're on. The Dexter SMS module accepts a phone number and a message. In order to tell the Dexter app that we want to send the URL from the bookmarklet step to the dexter sms step, we connect them. And that's it you're done! There are a few details we're leaving out and for the uber curious, here they are: <img src="/images/screenshots/05-rounded.png" class="image-shadow">  

  1. You'll notice that the phone number input on the Dexter SMS module is filled in. That's because it's in *user* mode, denoted by the radio buttons on the right panel. That means that the user will have to enter it when they subscribe to your app. If you always wanted to receive the message yourself, regardless of who clicked the button, you could put the input into text mode by clicking on the `A` radio button. Then you can type in your phone number. User mode will get a little clearer when we get to the use page. The little hand denotes 'map' mode which is how we mapped the URL to the Message field. 

  1. The left panel shows all the data you can grab. In this simplistic example we only have the data from the bookmarklet available, but as you add more steps to your apps you'll get to toggle where the source data is coming from. 
  
    <img src="/images/screenshots/07-rounded.png" class="image-shadow">  
  
1. **Use Your App** - Click the Use App button on the main nav and you'll get taken to your share page. This page is publically accessible and anyone can use your app, but you're the only one with the rights to edit it. Let's test it out. Click the Use This App button. Remember how we put the Phone Number into "user" mode, well, as a result, the wizard prompts us for our phone number. Go ahead and enter your phone number and hit finish. Once that's done, you'll see some new usage instructions for your app. Remember that bookmarklet that you dragged onto the canvas? Well, Dexter knows to generate a unique bookmarklet for every user and give them instructions to drag it onto their bookmarklet bar. Once you've dragged the button to your bookmarklet bar, give it a click - you should get a text message with a link to the page that you're on! 
  
    <img src="/images/screenshots/11-rounded.png" class="image-shadow">  

Congratulations on building your first app! You've just scratched the surface, want more? Read On! 

# What Is A Dexter App?

An app is simply a collection of JavaScript modules arranged in a meaningful way on the editor canvas. When we drag a module onto the canvas, we say that we are instantiating that module or forming a step in the application. 


## Steps

Steps are arranged by connecting one step to the next in the sequential order that data should be processed. In Dexter terms, we call This sequence a stream, which we cover in greater detail in the next section. 

There is one special step, called a **trigger step**. A trigger step is simply a way for the outside world to communicate with a Dexter app and kickoff some data processing. In its simplest form it is a URL that any external app can access via a `POST` request with arbitrary data that initiates the request. Dexter has some common triggers built-in (and we will add more regularly). Still, for any custom needs, you can always use the webhook trigger to pass in any type of data that you like. 

## Streams
A stream begins with a trigger step and is followed by one or more process steps. A stream does not have any formal termination point, but an execution of any given stream will complete when there are no more steps left. An app can have multiple streams.

## Module Types

There are three types of modules in Dexter:

<ol>
    <li>

<strong>Core modules</strong>

<br/>

Core modules are special. There are some cases where the simplistic behavior of modules presents a challenge to building a fully featured app. So we have carved out a space to deliver some magical methods to you. These will continue to expand, but here is what is available now: 

<br/>
<br/>

Queue - Sometimes while building a Dexter app you want to collect information that you're not ready to act on yet. Let's say you want to store a bunch of URLs from the bookmarklet trigger, but you don't want to do anything with them until a timer trigger is activated later. The queue step lets you stash that data for later use via the fetch step. The queue step takes a single parameter, "class_name", which is the name that you'll reference in your fetch step.  

<br/>
<br/>

Fetch - The fetch module is queues partner, it takes one parameter, "class_name", which tells Dexter which queued up data you want to retrieve. After a fetch step is executed all downstream steps will have access to the steps upstream of a queue with matching names. 

<br />
<br />

Switch - Sometimes in the course of assembling a stream you want to execute a different step depending on the evaluation of a condition. The switch step enables this by allowing you to specify up to 5 different conditions and branching a stream at a given point. 

    </li>

    <li>

<strong>Dexter modules</strong>

<br/>

These are the Dexter-blessed, built-in tools of the Dexter app ecosystem. They're available to every power user and developer in Dexter, and cover a broad range of behaviors.
    </li>

    <li>

<strong>User modules</strong>

<br/>

User modules are modules you've made for your own apps. You build them with the same tools we've used to build the vast library of Dexter modules, and you can use them the same in the App editor. The only difference between User modules and Dexter modules lies in distribution: only *you* can use your user modules.

    </li>

</ol>