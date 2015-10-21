# FAQ & Other Helpful Info

This section is very much a work in progress. We'll be adding to it as key questions become brought to our attention. If you have questions regarding anything that is not listed here, please visit our <a href="http://community.rundexter.com" target="_blank">community forums</a>, or connect with us on <a href="http://rundexter.slack.com" target="_blank">Slack</a>.

## How do I debug my app when it’s live?

Modules send messages to the Dexter logging component and those messages will appear in your browser’s developer console when your app is executing. In chrome, you should open up developer tools to view the messages; in firefox, you can use firebug or the native tools.

## What do the configuration icons mean?

You’ll notice some iconography and radio buttons just below the input titles on the right sidebar of your configuration panel. Each one is a different input method: 

   * **User Defined** - If this is selected, a user will define the input when they go to use the app. This happens during their configuration   
   * **Mapping** - If this is selected, the input will be whatever output you grab, drag, and drop from the output options, to the step below.
   * **Literal** - If this is selected, the input will be whatever you type into the provided field that will appear below the dropdown. NOTE: If you lead your input with a "=" the field can take raw javascript.
  
Each of these is useful depending on how your app is being used. In the SMS example we walked through to begin your Dexter experience, we set the Phone Number input to be user defined. If we went with literal instead, the number that would be used would be the one you typed in, meaning all links from the bookmarklet would be texted to that number.