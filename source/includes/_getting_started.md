# Getting Started

Dexter is a platform that makes it easy for developers to connect third-party APIs and build apps that automate your digital life. Everyday we use a handful of tools that do one thing and do it well, whether that’s ordering lunch or messaging the office a meme. However, more complex tasks require a combination of different solutions. Dexter is here to simplify connecting these services. With Dexter, you can connect the web the way you want.

## Detailed Walkthrough

The walkthrough gives you the basic instructions as you click through, but here's some more info in case you get stuck, confused, or just want to know more: 

1. **Create a New App** - Dexter is both for builders and consumers. We start with the building. Click the new app button on the top right of your screen. <img src="/images/screenshots/01-rounded.png" class="image-shadow">  

1. **Name Your App** - The tutorial autopopulates the title and description field with the details of the app we're going to build: a <a href="http://www.google.com/search?q=define:bookmarklet" target="_blank">bookmarklet</a> that sends a bookmarklet that sends the current url to your mobile phone via text message so that you can take it with you on the road! <img src="/images/screenshots/02-rounded.png" class="image-shadow">  

1. **Place Your First Trigger** - Triggers are actions performed in the outside world that tell Dexter to run the app. The way we notify the workflow about a trigger is to drag it onto the light gray bar at the top of the canvas. That’s our trigger bar. Think of it as the app's starting line. In this tutorial, our trigger is a <a href="http://www.google.com/search?q=define:bookmarklet" target="_blank">bookmarklet</a>. When clicked in a user's bookmark bar, it will send the current URL to their mobile device in an SMS. <img src="/images/screenshots/03-rounded.png" class="image-shadow">  

1. **Place Your First Module** - A module does the heavy lifting. It takes the data from the previous step(s) – in this case the bookmarklet – and acts on it. For our first app we want to send ourselves a text message. How is that done? First, we drag the Dexter SMS module onto the canvas. Next, drag the orange outlet on the bookmarklet to the input of the Dexter SMS step. <img src="/images/screenshots/04-rounded.png" class="image-shadow">  


1. **Connect The Data** - The last step is telling Dexter how the data moves in between steps. To do this, we click the configure button and bring up the configure panel. Each module has outputs and inputs. In this case the bookmarklet module outputs the URL of the page you're on. The Dexter SMS module accepts a phone number and a message. In order to tell the Dexter app that we want to send the URL from the bookmarklet step to the dexter sms step, we connect them. And that's it you're done! There are a few details we're leaving out and for the uber curious, here they are: <img src="/images/screenshots/05-rounded.png" class="image-shadow">  


  1. You’ll notice some radio buttons just below the input titles on the right sidebar. Here is where you can choose from an array of input methods: 
		  * **User Defined** - If this is selected, a user will define the input when they go to use the app. This happens during their configuration steps.
		  * **Mapping** - If this is selected, the input will be whatever output you grab, drag, and drop to the step. In this instance, the URL will be the message.
		  * **Literal** - If this is selected, the input will be whatever you type into the provided field that will appear below the dropdown. NOTE: If you lead your input with a "=" the field can take raw javascript.


  1. The left panel shows all the data you can grab. In this simplistic example we only have the data from the bookmarklet available, but as you add more steps to your apps you'll get to toggle where the source data is coming from. 
  
    <img src="/images/screenshots/07-rounded-v2.png" class="image-shadow">  
  
1. **Use Your App** - Click the Use App button on the main nav and you'll get taken to your share page. This page is publically accessible and anyone can use your app, but you're the only one with the rights to edit it. Let's test it out. </br></br>Click the “Use this App” button. This will take us through the configuration mode of our app. Remember how we set the Phone Number to be user provided when building the application? Well, as a result, the wizard prompts us for the phone number here. Go ahead and enter your phone number and hit “Finish.” Once that’s done, you’ll see some new usage instructions for your app. Drag the button to your bookmarklet bar and give it a click. You should get a text message with a link to the page that you’re on! 
  
    <img src="/images/screenshots/11-rounded.png" class="image-shadow">  

Congratulations on building your first app! You’ve just scratched the surface of what can be built on Dexter. Keep reading for information to help you build a more robust product. 

# What Is A Dexter App?

<img src="/images/illustrations/illustrated-editor.png" class="image-shadow">

A Dexter app is a collection of JavaScript modules arranged and connected in a meaningful way within the Dexter Editor. When we drag a module onto the canvas, we say that we are instantiating that module, or forming a *Step* in our application.

<img src="/images/illustrations/module-v-step.png">


## Steps

Steps are arranged by connecting one step to the next in the sequential order that data should be processed. In Dexter terms, we call this sequence a *Stream*, which we cover in greater detail in the next section.

<img src="/images/illustrations/stream-illustration.png" class="image-shadow">

One step stands out from the rest. This step is called the **Trigger step**. Triggers allow events outside of Dexter communicate with your app, and kick off a *Stream*. In its simplest form it is a URL that any external app can access via a `POST` request with arbitrary data that initiates the request. Dexter has some common triggers built-in (and we will add more regularly). Still, for any custom needs, you can always use the webhook trigger to pass in any type of data that you like. 

## Streams

A stream begins with a trigger step and is followed by one or more process steps. A stream does not have any formal termination point, but an execution of any given stream will complete when there are no more steps left. An app can have multiple streams, where each Stream is started by its own Trigger.

## Module Types

There are three types of modules in Dexter:


1. **Core modules** <br/>
   There are some cases where the simplistic behavior of modules present a challenge to building a fully featured app. Core modules provide some extra abilities that help you enhance basic modules . Dexter will continually expand its Core Module offering, but for now here is what is available (updated 10/20):  
   
			**_Queue_** - Sometimes while building a Dexter app you want to collect information that you’re not ready to act on yet. For example — Let’s say you want to store a bunch of URLs from the Bookmarklet Trigger, but you don’t want to do anything with them until a Timer Trigger is activated later. The Queue step lets you stash that data for later use. The Queue step takes a single parameter, `class_name`, which is the name that you’ll reference in your Fetch step when you are ready to use it.  
			
			**_Fetch_** - The fetch module is queues partner, it takes one parameter, `class_name`, which tells Dexter which queued up data you want to retrieve. After a fetch step is executed all downstream steps will have access to the steps upstream of a queue with matching names. 
			
			**_Switch_** - Sometimes in the course of building an app you want to execute steps conditionally. The switch step enables this by allowing you to specify up to 5 different conditions and branching a stream at a given point. 

1. **Dexter modules** <br/>
   These are the Dexter-blessed, built-in tools of the Dexter app ecosystem. They're available to every power user and developer in Dexter, and cover a broad range of behaviors.

1. **User modules** <br/>
   User modules are modules you've made for your own apps. You build them with the same tools we've used to build the vast library of Dexter modules, and you can use them the same in the App editor. The only difference between User modules and Dexter modules lies in distribution: only *you* can use your user modules.