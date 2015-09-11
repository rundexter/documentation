# SDK Overview

Dexter is a completely open system. The Dexter SMS module you dragged onto the canvas in the tutorial is open source and you can view it in our <a href="http://github.com/rundexter/dexter-sms" target="_blank">Github repository</a>. Anyone can contribute new modules via our client tools. We are working hard to expand our library with more and more third party integrations to minimize the amount of code every app developer needs to write. 

Want to help? Dexter apps are composed of an interconnected series of modules, each of which has a specific job to do. Together with the Dexter community, our goal is to build modules that cover the range of tasks that you might do with a third party provider -- like send a Slack message, add a song to a Spotify playlist, archive an item in Gmail, and more! 

## How it works

The Dexter SDK lets you create your own module, which is just a slightly enhanced Node.js library. You declare what kind of serializable input and output your module receives and sends, respectively.

When you're done, you push your code  into Dexter, using a custom git server. You can use either the sdk `dexter push` or a regular git program `git push dexter master` to send the code along.

Once it's in our system, it'll show up in the App editor as another module for you to use. You'll be able to wire it up however you want and run it alongside the built-in modules.

You can make and use as many custom modules as you like - the sky's the limit!

## Setting Up Your Environment

```shell
# install the Dexter client
$ npm install rundexter -g
```

To get started you need to have a working installation of <a href="https://nodejs.org" target="_blank">Node.js</a>. Dexter has been tested with `Node.js v0.12.5`, but it will likely work with other versions as well. 

## Log in
```shell
$ dexter login { your email }
# Enter your password when prompted
```
Before you do anything, you'll need to log in with your rundexter.com credentials. That lets us know who you are, so we can make the next steps happen.

## Make a key
```shell
$ ssh-keygen -f ~/.ssh/id_rsa -N ""
```

You’ll need to have an SSH key of some flavor (RSA or DSA). You might already have one - they’re useful things, and you’ve probably needed one before. They’re probably in ~/.ssh/ and called id_rsa and id_rsa.pub. If you don’t have one, make one now. You can run the given sample code to create a key, and [read more on the subject](https://help.ubuntu.com/community/SSH/OpenSSH/Keys) if you're interested.

## Send your key
```shell
# You can pass a path to a specific key here if you'd like
$ dexter add_key
# Make sure we got it!
$ dexter list keys
```

In order to be able to push your code to us, you’ll have to register your SSH key with us. We’ll use the default key if you have one, or you can give us a specific key you’d like you use. You can also register more than one key (on this machine or multiple machines) if you’d like.

# SDK Reference

## Authentication 
### dexter login
> Example

```shell
~/rundexter$ dexter login code@rundexter.com
Password: ****
Add your public key for GIT access? (yes) no
If you change your mind, you can always add it later
dexter add_key <keyname>(optional) Add an SSH key to the system

```
Parameter|Required|Description
---------|--------|-----------
email_required | true | Your email

Log into your rundexter.com account using the website's email and password.

Dexter remembers your credentials by adding a record to ~/.netrc, a fairly standard credential storage file used by FTP, Heroku, and a variety of other tools. Even though it's a plain-text file, don't worry!  We don't put your password in there, just an API key that only means something to the Dexter tools.

<aside class="notice">
This will always prompt for a password, which MUST be entered interactively.
</aside>

### dexter add_key
> Example: Default key

```shell
~/rundexter$ dexter add_key
Your key is now available on dexter!
```
> Example: Custom key

```shell
~/rundexter$ dexter add_key ~/dexter.rsa.pub
Your key is now available on dexter!
Make sure your .ssh/config is set to use this key for rundexter.com
```

Parameter|Required|Description
---------|--------|-----------
path | false |  Path to the key file

Add a public key file to the system. This file will be used to authenticate your access to the Dexter deployment servers when you publish code.

If left out, Dexter will first look for a file in ~/.ssh/id_rsa.pub, then in ~/.ssh/id_dsa.pub

Adding keys is useful if you want to have multiple people work on a single module, or if you work from multiple computers with varying security requirements (a work and a personal computer, for instance).

### dexter remove_key
> Example: Single match

```shell
~/rundexter$ dexter remove_key 3792y+
Matching keys removed: 1
```

> Example: Multiple matches (error)

```shell
~/rundexter$ dexter remove_key AAA
Request failed: More than one key was found that matches this query - please be more specific
```

Parameter|Required|Description
---------|--------|-----------
pattern | true |  A string pattern found in the key you want to delete

Remove a key from rundexter.com that you no longer want to have access to the Dexter deployment servers. The provided pattern can exist anywhere in the key, or it can be the entire key. Use the list_keys command if you need to figure out a good pattern to use, or pass in your machine name if you think it's unique.

We'll look for the pattern you provide in all of your registered keys, but we'll only delete a key if it's the only match we find. In case the pattern you give us matches more than one key, we'll ask you to be more specific instead of wiping them all out!

Removing a key doesn't prevent you from re-adding that key in the future.

### dexter list_keys
> Example

```shell
~/rundexter$ dexter list_keys
ssh-rsa AAA.........82Fregr= dexter@rundexter.com

ssh-dsa AAA.........38eeer2= dexter@rundexter.com
```

Show all the keys you've registered with rundexter.com

## Module Operations

### dexter check_name
> Example (available)

```shell
~/rundexter$ dexter check_name foobar
Package name is available
```

> Example (unavailable)

```shell
~/rundexter$ dexter check_name dexter-sms
Someone else owns this package already
```

Parameter|Required|Description
---------|--------|-----------
name | true |  The module name you're checking

Check to see if a given module name is taken. Dexter modules live in a global namespace, so if someone else has used a name, you can't also use it.

### dexter init
> Example (this folder)

```shell
~/rundexter$ dexter init
Initializing Dexter remote in /home/dexter/projects/my-dexter-module
Your dexter remote is ready
~/rundexter$ git remote -v
dexter  git@git.rundexter.com:/my-dexter-module (fetch)
dexter  git@git.rundexter.com:/my-dexter-module (push)
```

> Example (another folder)

```shell
~/rundexter$ dexter init /srv/app/dexter/module
Initializing Dexter remote in /srv/app/dexter/module
Your dexter remote is ready
~/rundexter$ git remote -v
dexter  git@git.rundexter.com:/server-dexter-module (fetch)
dexter  git@git.rundexter.com:/server-dexter-module (push)
```

Parameter|Required|Description
---------|--------|-----------
path | false |  The path to the git instance


Set up an existing git repository for use with the dexter command-line tool. So long as the target directory *looks* like a dexter module (i.e. it has valid package and meta json files), init will create a remote named "dexter" for you.

### dexter create
> Example

```shell
~/rundexter$ dexter create foobar
Initializing project as Dexter DocGuy (code@rundexter.com)
Created repo in /home/dexter/foobar
lodash@3.10.1 node_modules/lodash
Added .gitignore
Added README.md
Added fixtures/default.js
Added fixtures/env.example.js
Added form/README.md
Added form/checkout.form
Added form/run.form
Added form/settings.form
Added index.js
Added meta.json
Added package.json
Created initial commit [Oid 5e0536cebca0811079d4f3ec87f940839854cca4]
Initializing Dexter remote in /home/dexter/foobar
Your dexter remote is ready
```

Parameter|Required|Description
---------|--------|-----------
name | true |  The name of the new Dexter module

Create a skeleton for a new Dexter module. The following things happen:

1. The given name is first checked to make sure it's available
1. A module skeleton is copied and modified to match the given name
1. A local git repository is created for the module and its skeleton contents added as a first commit
1. The dexter remote is configured

### dexter run
> Example (default)

```shell
# runs fixtures/default.js
~/rundexter$ dexter run
{
    "foo": "bar"
}
```

> Example (custom)

```shell
# runs fixtures/barWithBaz.js
~/rundexter$ dexter barWithBaz
{
    "bar": "baz"
}
```

Parameter|Required|Description
---------|--------|-----------
name | true |  The name of the fixture to run

Execute the function using a built-in fixture. Every Dexter module is created with a default fixture which is used if you don't explicitly identify which fixture to run.

### dexter push
> Example

```shell
~/rundexter$ dexter push
Adding your module to Dexter...
Counting objects: 13, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (11/11), done.
Writing objects: 100% (13/13), 3.21 KiB | 0 bytes/s, done.
Total 13 (delta 0), reused 0 (delta 0)
---> Processing push
---> Sending to your environment
---> Push processed!
---> Success! Your module is now available in the Dexter Editor!
To git@git.beta.rundexter.com:/foobar
 * [new branch]      master -> master
```

Push your module to Dexter. This will update your code on our servers and deploy it for use by all Things and future Things in the system.

Note that deployment is largely tied to your module's version #. For example, let's say you do the following:

1. Create a module foobar at version 0.1
1. Add foobar to a Thing called Hello World.
1. Update foobar by with a new "baz" function and bump the version to 0.2
1. Add foobar to a Thing called Welcome Universe.

When this process is complete, "Hello World" will still be using foobar 0.1, and "Welcome Universe" will be using 0.2. You would have to explicitly update "Hello World" in order for it to start using foobar 0.2.