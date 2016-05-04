# Detailed Walkthrough

Our guided [walkthrough tutorial](https://rundexter.com/app/?tutorial=build) is the best way to get acquianted with Dexter, but here's some more info in case you get stuck, confused, or just want to know more:

1. **Create a New App** - Click the new app button on the top right of your screen. <img src="/images/screenshots/step-1.png" class="image-shadow">  

1. **Name Your App** - The tutorial autopopulates the title and description field with the details of the app we're going to build: a <a href="http://www.google.com/search?q=define:bookmarklet" target="_blank">bookmarklet</a> that sends the current url to your mobile phone via text message so that you can take it with you on the road! <img src="/images/screenshots/step-2.png" class="image-shadow">  

1. **Place Your First Trigger** - Triggers are actions performed in the outside world that tell Dexter to run parts of your app. The way we notify the workflow about a trigger is to drag it onto the light gray bar at the top of the canvas. That’s our trigger bar. Think of it as the app's starting line. In this tutorial, our trigger is a <a href="http://www.google.com/search?q=define:bookmarklet" target="_blank">bookmarklet</a>. When clicked in a user's bookmark bar, it will send the current URL to their mobile device in an SMS. <img src="/images/screenshots/step-3.png" class="image-shadow">

1. **Place Your First Module** - A module does the heavy lifting. It takes the data from the previous step(s) – in this case the bookmarklet – and acts on it. For our first app we want to send ourselves a text message. How is that done? First, we drag the Dexter SMS module onto the canvas. Next, drag the orange outlet on the bookmarklet to the input of the Dexter SMS step. <img src="/images/screenshots/step-4.png" class="image-shadow">  

1. **Connect The Data** - The last step is telling Dexter how the data moves in between steps. To do this, we click the configure button and bring up the configure panel. 
  <img src="/images/screenshots/step-5.png" class="image-shadow">  
  Each module has outputs and inputs. In this case the bookmarklet module outputs the URL of the page you're on. The Dexter SMS module accepts a phone number and a message as input. In order to tell the Dexter app that we want to send the URL from the bookmarklet step to the dexter sms step, we connect them. And that's it you're done! 
  <img src="/images/screenshots/step-6.png" class="image-shadow">  
  A few more notes about this screen and connecting your data:
  1. On the right sidebar, you'll see a list of all the inputs to the module. There are actually different ways to specify where each input should come from, and you can select each one by using the drop-downs. These ways are:
    <p>**Prompt the user** - If this is selected, when a user begins to use your app, they will be prompted for this input value. For example, you may prompt the user for the email address input in an email module if your app intends to email each user individually.</p>
    <p>**Use the visual mapping tool** - The input will be whatever output you grab, drag, and drop to the step. </p>
    <p>**Enter the value in manually** - The input will be whatever you type into the provided field. NOTE: If you lead your input with a `=` the field can take raw JavaScript.</p>
  1. The left panel shows all the data you can grab from upstream modules. In this simplistic tutorial we only have the data from the one bookmarklet module available, but as you add more steps and modules to your app, you can select data from any of these steps and use that as your input.
  
1. **Use Your App** - Click the Use App button on the main nav and you'll get taken to your share page. This page is publically accessible and anyone can use your app, but you're the only one with the rights to edit it. Let's test it out. </br></br>Click the “Use this App” button. This will take us through the configuration mode of our app. Remember how we set the Phone Number to be user provided when building the application? Well, as a result, the wizard prompts us for the phone number here. Go ahead and enter your phone number and hit “Finish.” Once that’s done, you’ll see some new usage instructions for your app. Drag the button to your bookmarklet bar and give it a click. You should get a text message with a link to the page that you’re on! 
  
    <img src="/images/screenshots/step-7.png" class="image-shadow">  

Congratulations on building your first app! You’ve just scratched the surface of what can be built on Dexter. Keep reading for information to help you build a more robust product. 

# What Is A Dexter App?

<img src="/images/illustrations/illustrated-editor.png" class="image-shadow">

A Dexter app is a collection of single-purpose modules arranged and connected in a meaningful way within the Dexter Editor. When we drag a module onto the canvas, we say that we are instantiating that module, or forming a *Step* in our application.

<img src="/images/illustrations/module-v-step.png">


## Steps

Steps are arranged by connecting one step to the next in the sequential order that data should be processed. In Dexter terms, we call this sequence a *Stream*, which we cover in greater detail in the next section.

<img src="/images/illustrations/stream-illustration.png" class="image-shadow">

One step stands out from the rest. This step is called the **Trigger Step**. Triggers allow events outside of Dexter (e.g., an RSS feed gets updated or a timer goes off) to communicate with your app, and kick off a *Stream*. Dexter has some common triggers built-in, and we will add more regularly. A single app can have multiple Triggers kick off different streams.

<aside class="notice">
For any custom needs outside Dexter's existing common Triggers, you can always use the Webhook Trigger to pass in any type of data that you like&mdash;it is simply a Dexter URL that any external app can access via a <tt>POST</tt> request with arbitrary data that initiates the request.
</aside>

## Streams

A Stream begins with a Trigger Step and is followed by one or more process steps. Each step will start only when the immediate prior step finishes and completes successfully. A stream does not have any formal termination point, but an execution of any given stream will complete when there are no more steps left. An app can have multiple streams, where each Stream is started by its own Trigger.

<aside class="notice">
Every step in a stream can reference and get data from <strong>all</strong> upstream steps, not just the immediate prior step. You can see all available upstream steps on the left hand side of the configure screen. For example, you can use the output from your Trigger Step as an input to the last step in your Stream.
</aside>

## Module Types

There are three types of modules in Dexter:

1. **System Modules** <br/>
   There are some cases where the simplistic behavior of modules present a challenge to building a fully featured app. System Modules provide some special abilities that help you enhance basic modules. Dexter will continually expand its Systen Module offering, but for now here is what is available:  
   
			**_Queue_** - Sometimes while building a Dexter app you want to collect information that you’re not ready to act on yet. For example — Let’s say you want to store a bunch of URLs from the Bookmarklet Trigger, but you don’t want to do anything with them until a Timer Trigger is activated later. The Queue step lets you stash that data for later use. The Queue step takes a single parameter, `class_name`, which is the name that you’ll reference in your Fetch step when you are ready to use it.  
			
			**_Fetch_** - The fetch module is queues partner, it takes one parameter, `class_name`, which tells Dexter which queued up data you want to retrieve. After a fetch step is executed all downstream steps will have access to the steps upstream of a queue with matching names. 
			
			**_Switch_** - Sometimes in the course of building an app you want to execute steps conditionally. The switch step enables this by allowing you to specify up to 5 different conditions and branching a stream at a given point. 

      **_Wait For Input_** - Wait for additional user input from the trigger.

1. **Standard Modules** <br/>
   These are standard modules that are built and maintained by the Dexter developer community. 

1. **User modules** <br/>
   User modules are modules you've made for your own apps. You build them with the [same tools](#sdk-tutorial-module-building) we've used to build the vast library of Dexter modules, and you can use them the same in the App editor. The only difference between User modules and Dexter modules lies in distribution: only *you* can use your user modules.
   
## Authentication & Authorization

Every Dexter app execution is associated with a Dexter user account. For anonymous-mode use cases (e.g., a Slack slash command), Dexter provides “Execute As” functionality. By choosing “Owner” from the dropdown, all URLs exposed by a Dexter app trigger will append a restricted api key that allows the executing user to invoke the app without having Dexter credentials. The user context, while the app is running, will be that of the app owner — the user who configured the app.

<img src="/images/screenshots/execute-as.png" class="image-shadow">

The API keys can be generated/regenerated from every User’s account page. 

## Code in the App Editor (Advanced)

Sometimes, you want to write a little custom Javascript to sanitize or otherwise deal with the inputs with a module. You can always do this by selecting the *Enter the value in manually* input option and prepending any Javascript you write with `=`. For example, if you want the input of a module to be a random number between 0 and 9, you can enter `=Math.floor(Math.random()*10)` as your input. We expose the `dexter` object to you to help you reference in-app variables or module outputs (see the [Module API reference](#dexter) for more details).

One common task you might want to do in the App Editor is reference upstream output data. Use `dexter`'s `step` function to specify the Step ID (found in the Properties explorer in the main editor view), and `output` to specify the output field. As a conrete example, to reference the output of the "channel ID" for a Slack slash command module, you can use:

<block class="highlight javascript">
 <code>
 =dexter.step('slackslash').output('channel_id').first()
 </code>
</block>

(we use `.first()` because all output is by default a collection of data)

### Code View

Sometimes though, you want to write *a lot* of custom Javascript into your app, and the tiny input box just won't cut it. This is where Dexter's built-in Code View comes in handy. You can write longer custom functions that you can use as the input to a module. Any code you write in Code View is global to your app and can be used in any modules. You reference your custom functions by calling `=app.myCustomFunction()` as the input to a module.
