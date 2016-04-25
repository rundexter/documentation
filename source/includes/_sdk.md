# SDK Installation & Overview

Dexter is a completely open system. Our modules are constructed from open tools, and all our module code is open source. Even the Dexter runtime is open to you.  Anyone can build and contribute new modules via our client tools.  Are we missing some functionality you need?  Create a completely new module and upload it to Dexter for your use.  Is one of our modules *almost*, but not exactly what you're looking for?  Fork the module, update its functionality, and upload it as your own.

Together with the Dexter community, our goal is to build modules that cover a broad range of integrations with third party services, like sending a Slack message, adding a song to a Spotify playlist, archiving an item in Gmail, and more!  If you've created a module that covers such a service, <a href="mailto:support@rundexter.com" target="_blank">let us know</a>, and we'll work with you to make it available to the community at large.

## How it works

The Dexter SDK along with the `dexter` command-line tool lets you create your own module, which is just a slightly enhanced Node.js library: it only requires a dedicated Dexter entry point and explicit documentation of inputs and outputs.  Otherwise you can follow your normal Node.js development process.

When you're done, you push your code into Dexter using our git server. You can send the code to Dexter via

`dexter push` or `git push dexter master` 

Once it's in our system, it'll show up in the App editor as another module for you to use in your “User Modules” panel. You'll be able to wire it up however you want and run it alongside the built-in modules, and there's no limit on how many Dexter modules you can create and use.

## Setting Up Your Environment

```shell
# install the Dexter client
$ npm install rundexter -g
```

To get started you need to have a working installation of <a href="https://nodejs.org" target="_blank">Node.js</a>. We have certified the Dexter client tools for use in recent versions up to `4.1.2`, however, the modules need to be executable in AWS Lambda’s <a href="http://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html" target="_blank">current Node.js infrastructure</a>.

When you're working with a compatible version of Node, use `npm install rundexter -g` to install the client.

## Log in
```shell
$ dexter login { your email }
# Enter your password when prompted
```
Before you do anything, you'll need to log in with your Dexter credentials. That lets us know who you are, so we can make the next steps happen.

## Make a key
```shell
$ ssh-keygen -f ~/.ssh/id_rsa -N ""
```

You’ll need to have an SSH key — either RSA or DSA. 
<aside class="notice">
You might already have one. They’re probably in ~/.ssh/ and called id_rsa and id_rsa.pub.
</aside>

If you don’t have one, make one now. You can run the given sample code to create a key, and [read more on the subject](https://help.ubuntu.com/community/SSH/OpenSSH/Keys) if you're interested.

## Send your key
```shell
# You can pass a path to a specific key here if you'd like
$ dexter add_key
# Make sure we got it!
$ dexter list_keys
```

In order to be able to push your code to us, you’ll have to register your SSH key with us. We’ll use your default key if you have one, or you can give us a specific key instead. You can also register more than one key (on this machine or multiple machines) if you’d like.
