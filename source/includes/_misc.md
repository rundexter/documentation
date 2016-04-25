# FAQ & Other Helpful Info

This section is very much a work in progress. We'll be adding to it as key questions become brought to our attention. If you have questions regarding anything that is not listed here, please visit our <a href="http://community.rundexter.com" target="_blank">community forums</a>, or connect with us on <a href="http://rundexter.slack.com" target="_blank">Slack</a>.

## What is Dexter?
Dexter is a platform that makes it easy for people to connect servicess across the web and build useful apps. The documentation you're reading now is meant for these **builders**. If you have questions about using a Dexter app, please get in touch with the creator of the app. We're also available on <a href="http://rundexter.slack.com" target="_blank">Slack</a> for support.

## How do I debug my Dexter app when it’s live?

Modules send messages to the Dexter logging component and those messages will appear in your browser’s developer console when your app is executing. In Chrome, you should open up developer tools to view the messages; in Firefox, you can use Firebug or the native tools.

## What does "Prompt the user", "User the visual mapping tool", and "Enter the value in manually" mean?

You’ll notice some iconography and radio buttons just below the input titles on the right sidebar of your configuration panel. Each one is a different input method: 

   * **Prompt the user** - If this is selected, a user will define the input when they go to use the app. This happens during their configuration   
   * **Use the visual mapping tool** - If this is selected, the input will be whatever output you grab, drag, and drop from the output options, to the step below.
   * **Enter the value in manually** - If this is selected, the input will be whatever you type into the provided field that appears below the dropdown. NOTE: If you lead your input with a `=` the field can take raw Javascript.
  
Each of these is useful depending on how your app is being used. In the SMS example we walked through to begin your Dexter experience, we set the Phone Number input to be user defined. If we went with literal instead, the number that would be used would be the one you typed in, meaning all links from the bookmarklet would be texted to that number.

## What does it mean when the Properties explorer says "New Version - Click to Update"?

Sometimes module developers will need to update their module&mdash;to add features, fix a bug, etc. A new version alert is triggered whenever a new module is deployed and the version number (in package.json&mdash;see the [Module API reference](http://localhost:4567/#module-api-reference)) is incremented by the developer.