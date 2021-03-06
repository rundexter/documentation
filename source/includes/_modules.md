# Module API Reference

Modules in Dexter are the building blocks you'll use to assemble your Apps. They're expressions of the principal of doing one thing and doing it well: each is designed to have a single, clear task that works independently of the rest of your code. It's the job of the App to decide how these behaviors interact - the modules themselves only care about what kind of input they get and emitting consistent output.

<aside class="success">
Modules are just specialized Node.js libraries with a bit of special sauce in the form of extra metadata, baked-in tests, and an SDK to help speed development.
</aside>

## Anatomy of a module

Only 3 files are mandatory for a Dexter module:

> The Entrypoint (index.js)

```javascript
module.exports = {
    run: function(step, dexter) {
        this.complete();
    }
};
```

- **index.js**: This is the main touchpoint for your application. At a minimum, it needs to export a single function `run(step, dexter)` and call `this.complete()`. We'll go into more detail about this later on.

<div></div>

> The Node.js Module Descriptor (package.json)

```json
{
  "name": "my_dexter_module",
  "description": "A minimal example Dexter module",
  "version": "0.0.1"
}
```

- **package.json**: This is a regular [Node.js package definition](https://docs.npmjs.com/files/package.json). This lets you use a Dexter module just like you would any other Node.js package.

<div></div>

> The Dexter Descriptor (meta.json)

```json
{
    "title": "My Dexter Module",
    "icon": "heart",
    "inputs": [{ 
        "id": "url",
        "title": "Page URL"
    }, { 
        "id": "term",
        "title": "Search term"
    }],
    "outputs": [{ 
        "id": "links",
        "title": "Found links"
    }],
    "providers": [{
        "name": "github",
        "scopes": "public_repo,user:email,read:org"
    }]
}
```

- **meta.json**: Special definitions used by Dexter are stored here.
    * **title (string, required)**: The user-friendly title of your module
    * **icon (string, optional)**: A [FontAwesome icon](http://fortawesome.github.io/Font-Awesome/cheatsheet/) that will show up alongside your module in the App editor.
    * **inputs (array, optional)**: Definitions for any inputs your module accepts. Each item in the array should have 2 or 3 properties:
        * *id (string)*: The input's code-friendly key (ex: first_name)
        * *title (string)*: The input's user-friendly name (ex: First Name)
        * *selector_type (string, optional)*: The default configuration setting for the input: either **literal** (a plain-text string should be used), **map** (it should be wired up to the output of another module), or **user** (it should be entered by the user of the app).  The default is **map**.
    * **outputs (array, optional)**: Definitions for any outputs your module generates.
        * *id (string)*: The input's code-friendly key (ex: first_name)
        * *title (string)*: The input's user-friendly name (ex: First Name)
    * **providers (array, optional)**: A list of Dexter OAuth providers that your module relies on
        * *name (string)*: The lower-case name of the [provider](#providers) that you're using
        * *scopes (string)*: Provider scopes required by the module

## The runtime wrapper
```javascript
module.exports = {
    run: function(step, dexter) {
        //See if we left off from a previous run
        var lastFoo = step.global('lastFoo', null),
            thisFoo = lastFoo || 0,
            foos = step.input('foo');
        this.log('Looked for cached foo: :lastFoo, starting at :thisFoo',
            { lastFoo: lastFoo, thisFoo: thisFoo });
        //Have we gone too far?
        if(thisFoo > foos.length) {
            return this.fail('Not enough foo!');
        }
        //Do our thing
        this.runFoo(foos[thisFoo]);
        thisFoo ++;
        //Are we done yet?
        if(thisFoo === foos.length) {
            return this.complete();
        }
        //If not, let's go again!
        dexter.setGlobal('lastFoo', thisFoo);
        this.replay();
    }
}
```

Before dexter modules are run, they're wrapped up by a [BaseModule](#basemodule) class that provides the tools you need to integrate your code into Dexter Apps.

That means if you write a class that exports something like:

`{ getData: fn, run: fn }`

Once it gets inside the Dexter App, it ends up looking like:

`{ getData: fn, run: fn, log: fn, complete: fn, fail: fn, log: fn }`

This wrapping process lets you focus on the details of your module — and even run your module outside of Dexter — without worrying about managing Dexter’s dependencies.

<aside class="warning">
If your module exports properties named complete, fail, log, or run, your module will not work!
</aside>

## Inputs

As mentioned above, the meta.json file defines a collection of inputs expected by a module. To retrieve an input, you can use the `step.input(<input_id>)` from the run method. Calling step.input returns data wrapped up by a DexterCollection. For inputs that are expected to have a single value, we provide a convenience method `DexterCollection.first()` that will return the first item in the collection. Conversely, `DexterCollection.each(callback)` allows easy iteration over the collection. DexterCollections can also be inspected via standard array accessors.

## Outputs

```javascript
module.exports = {
  run: function() {
     this.complete({
        message: "This is the first" 
     });
  }
}
```

As mentioned above, the meta.json file defines a collection of outputs your module plans to expose. Your module should provide its outputs to Dexter by calling `this.complete(<object|array>)` from the scope of the run method. 

<div></div>

```javascript
module.exports = {
 run: function() {
    this.complete([{
        message: "This is the first message"
    },{
        message: "This is the second message"
    }]);
 }
}
```

If appropriate, returning a collection of objects is also acceptable, downstream modules are expected to handle arrays of data just like your modules!

<div></div>

You can read more about outputs under [BaseModule::complete](#basemodule-complete-data).


## Data collections

Calls to `step.input` and `step.output` return data wrapped up by a DexterCollection, which makes managing an uncertain amount of data easier.

Function|Description
---------|--------
first() | Gets the first item in the collection, or null if the collection is empty
toArray() | Exports the entire collection as a native Array, which will be empty if there's no underlying data
each(callback) | Executes callback against each item in the collection using [lodash's forEach](https://lodash.com/docs#forEach)


Property|Description
---------|--------
length | How many items are in the collection
[0...length-1] | Access the collection like an array, returns `undefined` if the index doesn't exist.

## Module functions
When you create a module, it implicitly extends a Dexter BaseModel before it's run. The BaseModel provides several functions that are required for your Module to run in a Dexter App.

### BaseModule.run(step, dexter)

Parameter|Type|Description
---------|--------|-----------
[step](#step) | DexterStep | Contextual information for the module (inputs, etc.)
[dexter](#dexter) | DexterApp | Global context for the whole App (runtime variables, configuration, etc.)

This is the main entry point to your module. By the time this has been run, Dexter has assembled your inputs based on the App developer's configuration and made them available to you via `[step](#step)`. If you need broader contextual information, you can all the data that's been generated for and by the app using `[dexter](#dexter)`.

<aside class="warning">
Dexter calls this function automatically when it's your module's turn to run in the App.  It should *not* be called again - you'll create a difficult-to-debug recursion.
</aside>

### BaseModule.complete(data)
> No output

```javascript
return this.complete();
```

> Single output

```javascript
return this.complete({ foo: 'Foo', bar: 'Bar' });
```

> Multiple outputs

```javascript
return this.complete([
    { foo: 'Foo', bar: 'Bar' },
    { foo: 'Baz', bar: null }
]);
```

Parameter|Type|Description
---------|--------|-----------
data | array,object | An object containing your outputs, or an array of objects containing your outputs if necessary

When your module has successfully finished its job, it needs to call `complete()` to let Dexter know it's safe to continue the current App.

If you have any useful information you want to make available to other modules, you'll want to return it here. Dexter assumes that you're returning an object or array, and that the object only has properties that match those you've defined in your `[meta.json:output](#anatomy-of-a-module)`. If you return a collection, each object's properties are correctly configured, Dexter will still know how get and share the data with other modules.

<aside class="warning">
Note that you should only call `complete` once, and after you do, the process will exit. Calling `return this.complete()` as shown in the code samples is one way to guarantee that you only make one call and don't cause a race condition afterwards, though you can use careful code flow to do the same.
</aside>

<aside class="success">
<p>Your response object properties can be any serializiable value.</p>
<p>Keep in mind, however, that a lot of Dexter's power comes from the ability of unrelated modules to be able to communicate with one another. If you output a property that's an object, array, or other complex type, other modules will need to know how to handle that *specific* type in order to work with your module. Returning only basic values like strings or numbers allows your module to be m uch more flexible and useful in the Dexter ecosystem.</p>
</aside>

### BaseModule.fail(details)
> Simple error

```javascript
return this.fail('Something bad  happened');
```

> Detailed error

```javascript
return this.fail({ message: 'Something bad happened', badthing: { foo: 'bar' }});
```

> Error forwarding

```javascript
try {
  foo.bar();
} catch(e) {
  return this.fail(e);
}
```

Parameter|Type|Description
---------|--------|-----------
error | string,Error,object | Details about what went wrong

When something goes badly wrong, `fail()` lets Dexter know to abort the app.

Every call to `fail()` automatically gets logged, and if you pass an error as the first argument, any available call stack will be logged as well.

<aside class="warning">
Note that you should only call `fail()` once, and after you do, the process will exit. Calling `return this.fail()` as shown in the code samples is one way to guarantee that you only make one call and don't cause a race condition afterwards, though you can use careful code flow to do the same.
</aside>

### BaseModule.log(details)
> Simple log

```javascript
this.log('Querying Foo...');
```

> Log with data details

```javascript
this.log('Back from foo', { message: "bar", foo: foo.results() });
```

The Dexter logger is a powerful tool that lets you send messages from your app straight to your browser's javascript console.  At its simplest, the logger lets you send text to your console from any point in your module via `this.log('Hello!')`.  Even this basic usage gives you a wealth of debugging data - when you see the log inside your browser console, that "Hello" is followed by the time the log was run and a complete copy of the application's data at the point your log was made, including input/output data, the current step, user data, and more!

If you've got some specific information you want to keep track of, you can pass it along as well. Just pass an object containing all the additional information you'd like to capture into the logger's second parameter, like `this.log('Hello!', { who: 'World' })`.  Remember that a serialized copy of the App is included for free with each log message — there's no need to duplicate its information in your log data.

The best part about the logger is that it's output is persistent.  If your app encounters an error, you can replay the logs from its last run by calling `dexter.instance()` in your browser's console in either the App editor or on the App's homepage.  You can also see your recent history by calling `dexter.recent()`, then dig into a specific historical instance by calling `dexter.instance(instance_id)`.

### BaseModule.replay()
```javascript
var lastIndexKey = step.config('id') + '_lastIndex',
    lastIndex = dexter.global(lastIndexKey, 0),
    data = step.input('data'),
    loopTo = lastIndex + 5,
    i;
if(loopTo > data.length) loopTo = data.length;
for(i = lastIndex; i < loopTo; i ++) {
    this.processData(data[i]);
}
if(loopTo === data.length) {
    this.complete();
} else {
    dexter.setGlobal('lastIndex', loopTo);
    this.replay();
}
```

One of the downsides of working with APIs is that many of them are rate limited. This poses a problem if you're trying to handle an arbitrary amount of data in the 60s time window allotted to a Dexter module.  If you're in a situation where you're not sure if you can handle all the data you've been given, `replay()` can help you out.

Calling replay will rerun the current module with the exact same inputs it was originally given. By using the App's global state management tools, your module can keep running until it finishes what it set out to do.

<aside class="warning">
There is currently a 50-step limit built into Dexter. That's more than enough for most Apps, *unless* you're relying heavily on replay to accomplish your goals. Please <a href="mailto:support@rundexter.com">contact us</a> if you would like to increase your limit.
</aside>

## App data
### dexter
This is the 2nd parameter passed into your module's `run()` function, and represents a high-level view into all that's gone on in your app so far.

It's also the accessor you use to map inputs and add switch cases when wiring up your App - more on that in the App UI section!

### dexter.instance(key, default)
```javascript
var url = 'http://mylogger.example.com/?msg=Hello&from='
    + dexter.instance('instance_id', '(unknown)');
```
Fetch some runtime information about this *instance* of an App. 

Key|Type|Description
---------|--------|-----------
instance_id | string | A unique id for this particular run of the App. Useful if you want track data on a per-run basis.
active_step | string | The ID of the step that's currently running. Inside your module, this is the same as step.config('id')


### dexter.environment(key, default) 
```javascript
if(!dexter.environment('slack_token')) {
    return this.fail('A slack_token environment variable is required for this module');
}
var slack = new Slack(dexter.environment('slack_token'));
```

Fetch any private App variables that have been registered.

### dexter.user(key, default)
```javascript
var email = step.input(email, dexter.user('email'));
```

Basic information about the user who's running the App

Key|Type|Description
---------|--------|-----------
profile.id | int | The Dexter system ID for the user
profile.email | string | The user's email
profile.image | string | A fully qualified URL pointing to the user's profile image
profile.api_key | string | The API key used by the user to access Dexter services (SMS, SMTP, etc.)


### dexter.app(key, default)
```javascript
var log = new ParseLog({ msg: 'Module start', app: dexter.app('id', '(unknown)') });
log.save();
```

Details about the App itself. Details about the *instance* of the App being run come from `instance()`.

Key|Type|Description
---------|--------|-----------
id | string | Dexter system ID for the workflow
title | string | User-friendly title 
description | string | Longer description about what the App does


### dexter.global(key, default)
```javascript
var cacheKey = step.module('name') + '_cache',
    cache = dexter.global(cacheKey, null);
if(!cache) {
    cache = this.calculate();
    dexter.setGlobal(cacheKey, cache);
}
this.complete({ data: cache });
```

Fetch globally stored information assigned during the use of this app.

### dexter.globals()
```javascript
var globalKeys = Object.keys(dexter.globals()),
    myKey = 'aKeyIWantToUse';
if(globalKeys.indexOf(myKey) !== false) {
    return this.fail('Global key ' + myKey + ' already in use!');
}
```

Get the entire collection of in-use global data in a single object.


### dexter.setGlobal(key, value)
```javascript
var cacheKey = step.module('name') + '_cache',
    cache = dexter.global(cacheKey, null);
if(!cache) {
    cache = this.calculate();
    dexter.setGlobal(cacheKey, cache);
}
this.complete({ data: cache });
```

An App-wide data store for the entire instance of the app. This is the ONLY place you can store information explicitly between steps and modules. Any serializable data can be assigned to a global. 

### dexter.provider(name)
```javascript
var credentials = dexter.provider('github').credentials(),
    GitHubApi = require('github'),
    github = new GitHubApi({ version: '3.0.0' });
github.authenticate({
    type: 'oauth',
    token: credentials.token
});
this.log('Authenticated', dexter.provider('github').data('username');
```

If your module has identified a need for providers in [meta.json](#anatomy-of-a-module), Dexter will give you the basic information you need to integrate with that provider on behalf of the App's user.  Take a look at the [provider settings](#providers) for details.

## Step

This is the first parameter passed to your module's `run()` function, and probably the most important. It contains both the configuration of your module for the current App as well as the data that's been assigned to it.

### step.config(key, default)
```javascript
var stepId = step.config('id'),
    instanceId = dexter.instance('id'),
    storageKey = instanceId + '.' + stepId;
var record = new ParseLog({ msg: 'Module start', src: storageKey });
log.save();
```

Fetch configuration information about this module at its current spot in the workflow.

Key|Type|Description
---------|--------|-----------
id | string | A unique ID for the step (related to but not guaranteed to be the same as the module name)
next | object | A collection of information about the possible next steps after this one
title | string | A human-friendly name for the step
type | string | What kind of step it is - currently either 'trigger' or 'module'
type_name | string | The specific step type, based on type. For modules, this is the module name.

### step.prev()
```javascript
var prev = step.prev();
if(prev.module('name') === 'foo') {
    return this.fail('Cannot run foo before bar. That would be inconceivable.');
}
```

Get the step that ran before this one.

### step.trigger()
```javascript
var trigger = step.trigger();
if(trigger.config('type_name') == 'timer' && this.lastRan() > moment().subtract(1, 'hours')) {
    return dexter.global('my_cache');
} else {
    return this.liveData();
}
```

Get the trigger step upstream from this step.

### step.module(key, default)
```javascript
var name = step.module('package.name', '(unknown)')
    , title = step.module('meta.title', '(unknown)')
    , text = 'Run from ' + title + ' (' + name + ')';
var record = new ParseLog({ msg: text });
```

Gets information about the module associated with the step.

Key|Type|Description
---------|--------|-----------
git_url | string | A fully-qualified URL to the primary git repository for this module
is_global | integer | "1" if this is a Dexter module, "0" if this is a user module
meta.* | misc | Access to any of the data in the module's meta.json file
package.* | misc | Access to any of the data in the module's package.json file
name | string | Shortcut for "package.name"
title | string | Shortcut for "package.title"
type | string | Either "global" for Dexter modules or "user" for user modules.
created_at | string | Date when the module was first made
updated_at | string | Date when the module was last updated in Dexter
user_id | int | User ID of the module's maintainer

### step.input(key, default)
```javascript
var email = step.input('email', 'default@example.com').first(),
    messages = step.input('message'),
    message = messages.toArray().join('<br>');
this.queueEmail(email, 'You have messages', message);
return this.complete();
```

Fetch an input bound to this step in the App editor.

Unlike most other functions, this returns a [Dexter collection](#data-collections) object instead of a raw value. This wrapper helps you navigate multiple or missing inputs as you prepare to run your module. You can read more about inputs [here](#inputs)

### step.inputs()
```javascript
var inputs = step.inputs(),
    inputKeys = Object.keys(inputs),
    types = {};
inputKeys.forEach(function(key) {
    types(key) = typeof inputs[key];
});
console.log(types);
//inputs: { x: 1, y: [2,3], z: null }
//log: { x: integer, y: array, z: null }
```

Get the raw input object. This will be a single object with all the input properties defined in your `meta.json:inputs` section and the raw values as parsed by Dexter.

### step.outputs()
```javascript
var outputs = step.outputs(),
    outputKeys = Object.keys(outputs),
    types = {};
outputKeys.forEach(function(key) {
    types(key) = typeof outputs[key];
});
console.log(types);
//outputs: { x: 1, y: [2,3], z: null }
//log: { x: integer, y: array, z: null }
```

Get the raw output object. This will be a single object with all the output properties emitted by the module, which should match the outputs defined in that module's `meta.json:outputs` section.

## Providers

Dexter currently supports the following API service providers:

* [asana](https://github.com/rundexter/documentation/blob/master/source/providers/Asana.md)
* [bitbucket](https://github.com/rundexter/documentation/blob/master/source/providers/BitBucket.md)
* [bitly](https://github.com/rundexter/documentation/blob/master/source/providers/Bitly.md)
* [evernote](https://github.com/rundexter/documentation/blob/master/source/providers/Evernote.md)
* [facebook](https://github.com/rundexter/documentation/blob/master/source/providers/Facebook.md)
* [flickr](https://github.com/rundexter/documentation/blob/master/source/providers/Flickr.md)
* [github](https://github.com/rundexter/documentation/blob/master/source/providers/GitHub.md)
* [google](https://github.com/rundexter/documentation/blob/master/source/providers/Google.md)
* [instagram](https://github.com/rundexter/documentation/blob/master/source/providers/Instagram.md)
* [linkedin](https://github.com/rundexter/documentation/blob/master/source/providers/LinkedIn.md)
* [mailchimp](https://github.com/rundexter/documentation/blob/master/source/providers/Mailchimp.md)
* [runkeeper](https://github.com/rundexter/documentation/blob/master/source/providers/RunKeeper.md)
* [slack](https://github.com/rundexter/documentation/blob/master/source/providers/Slack.md)
* [spotify](https://github.com/rundexter/documentation/blob/master/source/providers/Spotify.md)
* [trello](https://github.com/rundexter/documentation/blob/master/source/providers/Trello.md)
* [tumblr](https://github.com/rundexter/documentation/blob/master/source/providers/Tumblr.md)
* [twitter](https://github.com/rundexter/documentation/blob/master/source/providers/Twitter.md)


When you ask for provider information via [dexter.provider(name)](#dexter-provider-name), you're given a Provider object with the following methods:

### provider.token()
```javascript
var token = dexter.provider('myapi').token(),
    lib = require('myapi'),
    data = lib.query('/some/endpoink', { token: token.access_token, client: token.client_id });
```

Get the access token for the App's user.

<aside class="warning">
If you're testing using the built-in Dexter key/secret (i.e. you're not using your own), you <strong>will not</strong> be given the client/customer secret.
</aside>

For OAuth2 providers you'll get:

<block class="highlight javascript">
  <code>
  {<br>
    access_token: "...",<br>
    client_id: "..."<br>
    <em>client_secret: "..."</em> //Not included with global apps!<br>
  }
  </code>
</block>

OAuth1 providers get:

<block class="highlight javascript">
 <code>
 {<br>
    access_token: "...",<br>
    access_token_secret: "...",<br>
    consumer_key: "...",<br>
    <em>consumer_secret: "..."</em> //Not included with global apps!<br>
 }
 </code>
</block>

### provider.data(key, default)
```javascript
var username = dexter.provider('github').data('username');
```

Get pre-cached data from the user's provider information.  See the individual provider documentation for details.

## Testing modules

You don't have to repeatedly deploy your module to Dexter in order to make sure it works: test your module with `dexter run`. The Dexter SDK comes with a simple test framework that allows you to set up sample data sets (fixtures) that describe situations your module might have to deal with.

For example, you could make one fixture describing a single input value and another with multiple inputs.  You could also have a third with no or broken data to see how your module deals with error states.  Later, as you collect feeback on your module, you could add fixtures for other use cases you might not have considered during your initial development - or ask other users to add  them for you!

### Structure of a fixture
```javascript
module.exports = {
    data: {
        local_test_step: {
            input: {
                foo: 'bar',
                hello: [ 'world' ]
            }
        }
    },
    user: {
        providers: {
            dexter: {
                credentials: {
                    access_token: '123abc'
                }
            }
        }
    }
};
```

If you used the SDK to create your module, you've already got one fixture set up in your environment called "default" (in fixtures/default.js).  It contains a mockup of the data in a Dexter App that can pertain to your module.  Most modules only care about 2 sections: "data" and "user".

The `user` object is primarily important in providing test credentials to applications that rely on [providers](#providers).  `user.providers.[provider].credentials` should hold either a valid access_token for OAuth2 providers or the access_token, access_token_secret, consumer_key, and consumer_key_secret for OAuth1 providers (see the individual provider documentation for details).

The `data` object defines the data coming into your module via `data.local_test_step.input`.  This input object can have a key for each of the inputs you identify in your [meta.json](#anatomy-of-a-module).  For example, an emailer might have:

<block class="highlight javascript">
  <code>
  input: {<br>
    &nbsp;&nbsp;to: [ "foo@bar.com", "bar@baz.com" ],<br>
    &nbsp;&nbsp;from: "hello@world.com",<br>
    &nbsp;&nbsp;body: "Hello there!"<br>
  }
  </code>
</block>

### Private data in fixtures

> default.js

```javascript
var env = require('./env')
module.exports = _.merge({
    user: {
        providers: {
            dexter: {
                credentials: {
                    //See env.js
                },
                data: {
                    username: 'foo'
                }
            }
        }
    }
}, env);
```

> env.js

```javascript
module.exports = {
    user: {
        providers: {
            dexter: {
                credentials: {
                    access_token: 'abc123'
                }
            }
        }
    }
};
```

It's a good idea to put your fixtures in version control - that way other Dexter users can use and expand on the tests you've created.  However, it's **never** a good idea to put real user data into version control, particularly in a public ecosystem!

The Dexter SDK provides a secure data isolation strategy out of the box.  Your default fixture automatically extends itself with data found in `fixtures/env.js` - a file which, by default, is *not* added to your git repository.  That means you can safely use it to store your personal test data: emails, access tokens, phone numbers, etc.  You can then re-use this data in all your other fixtures just by using the same `_.merge` command found in `fixtures/default.js`.

### Using multiple fixtures

All you need to do to add new fixtures to your module is to copy `fixtures/default.js` to a new file and change the new file's contents.  For example, let's say we wanted to test what happens in our mailer module when no "from" email is provided.  We would just

<block class="highlight shell">
  <code>
  cp fixtures/default.js fixtures/no_from.js<br>
  # Set data.local_test_step.inputs.from = ''<br>
  dexter run no_from
  </code>
</block>
