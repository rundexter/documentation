# SDK Reference

## Authentication 
### dexter login
> Example

```shell
$ dexter login code@rundexter.com
Password: ****
Add your public key for GIT access? (yes) no
If you change your mind, you can always add it later
dexter add_key <keyname>(optional) Add an SSH key to the system
```

Parameter|Required|Description
---------|--------|-----------
email_required | true | Your email

Log into your Dexter account using your email and password.

### dexter add_key
> Example: Default key

```shell
#You can pass a path to a specific key here if you'd like
$ dexter add_key
Your key is now available on dexter!
#Make sure we got it
$ dexter list_keys
```

> Example: Custom key

```shell
$ dexter add_key ~/dexter.rsa.pub
Your key is now available on dexter!
Make sure your .ssh/config is set to use this key for git.rundexter.com
```

Parameter|Required|Description
---------|--------|-----------
path | false |  Path to the key file

Add a public ssh key file to the system. This file will be used to authenticate your access to the Dexter deployment servers when you publish code.

If left out, Dexter will first look for a file in `~/.ssh/id_rsa.pub`, then in `~/.ssh/id_dsa.pub`.

### dexter remove_key
> Example: Single match

```shell
$ dexter remove_key 3792y+
Matching keys removed: 1
```

> Example: Multiple matches (error)

```shell
$ dexter remove_key AAA
Request failed: More than one key was found that matches this query - please be more specific
```

Parameter|Required|Description
---------|--------|-----------
pattern | true |  A string pattern found in the key you want to delete

Remove a key from Dexter. The provided pattern can exist anywhere in the key, or it can be the entire key. Use the `dexter list_keys` to view your keys.


### dexter list_keys
> Example

```shell
$ dexter list_keys
ssh-rsa AAA.........82Fregr= dexter@rundexter.com

ssh-dsa AAA.........38eeer2= dexter@rundexter.com
```

Show all the keys you've registered with rundexter.com

## Module Operations

### dexter init
> Example (this folder)

```shell
$ dexter init
Initializing Dexter remote in /home/dexter/projects/my-dexter-module
Your dexter remote is ready
$ git remote -v
dexter  git@git.rundexter.com:/my-dexter-module (fetch)
dexter  git@git.rundexter.com:/my-dexter-module (push)
```

> Example (another folder)

```shell
$ dexter init /srv/app/dexter/module
Initializing Dexter remote in /srv/app/dexter/module
Your dexter remote is ready
$ git remote -v
dexter  git@git.rundexter.com:/server-dexter-module (fetch)
dexter  git@git.rundexter.com:/server-dexter-module (push)
```

Parameter|Required|Description
---------|--------|-----------
path | false |  The path to the git instance


Set up an existing git repository for use with the dexter command-line tool. So long as the target directory *looks* like a dexter module (i.e. it has valid package and meta json files), init will create a remote named "dexter" for you.

This gets called automatically when you `dexter create` a module, but is useful if you want to enable an existing module to work with Dexter.  In that case, you need to add a `meta.json` file before you can run `dexter init`.

### dexter create
> Example

```shell
$ dexter create foobar
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

    1. A local git repository is created for the module with some scaffolding code added as a first commit
    
    2. The dexter remote is configured

Note that your code won't show up in the Dexter App Editor until you `dexter push`.

### dexter run
> Example (default)

```shell
# runs fixtures/default.js
$ dexter run
{
    "foo": "bar"
}
```

> Example (custom)

```shell
# runs fixtures/barWithBaz.js
$ dexter run barWithBaz
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
$ dexter push
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

Push your module to Dexter. This will update your code on our servers and deploy it for use by your Dexter App Editor.

<aside class="notice">You must first commit any changes in your working directory using “git commit” before you can push to Dexter.</aside>

Deployment is tied to your module’s version number. For example, let’s say you do the following:

    1. Create a module foobar at version 0.1
    
    2. Add foobar to an App called Hello World.
    
    3. Update foobar by with a new "baz" function and bump the version to 0.2

    4. Add foobar to an App called Welcome Universe.

When this process is complete, "Hello World" will still be using foobar 0.1, and "Welcome Universe" will be using 0.2. You would have to explicitly update "Hello World" in order for it to start using foobar 0.2.
