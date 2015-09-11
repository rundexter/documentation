# Module API 

Modules in Dexter are the building blocks you'll use to assemble your Apps. They're expressions of the principal of doing one thing and doing it well: each is designed to have a single, clear task that works independently of the rest of your code. It's the job of the App to decide how these behaviors interact - the modules themselves only care about what kind of input they get and emitting consistent output.

<aside class="success">
Modules are just specialized Node.js libraries with a bit of special sauce in the form of extra metadata, baked-in tests, and an SDK to help speed development. Most can be used independently of Dexter, and can be incorporated into any other Node.js or [Browserified](http://browserify.org/) application.
</aside>

## Anatomy of a module

Only 3 files are mandatory for a Dexter module:

> The Entrypoint (index.js)

```javascript
module.exports = {
    run: function(step, dexter) {
        this.succeed();
    }
};
```

- **index.js**: This is the main touchpoint for your application. At a minimum, it needs to export a single function "run(step, dexter)" and call "this.succeed()". We'll go into more detail about this later on.

<div></div><!--clear code-->

> The Node.js Module Descriptor (package.json)

```json
{
  "name": "my_dexter_module",
  "description": "A minimal example Dexter module",
  "version": "0.0.1"
}
```

- **package.json**: This is a regular [Node.js package definition](https://docs.npmjs.com/files/package.json). This lets you use a Dexter module just like you would any other Node.js package - you can npm install --save to your hearts content, add your module to a public or private NPM server, and otherwise distribute it outside of Dexter.

<div></div><!--clear code-->

> The Dexter Descriptor (meta.json)

```json
{
    "title": "My Dexter Module",
    "icon": "heart",
    "inputs": [{ 
        "id": "url",
        "title": "Page URL", 
        "type": "string" 
    }, { 
        "id": "term",
        "title": "Search term",
        "type": "string" 
    }],
    "outputs": [{ 
        "id": "links",
        "title": "Found links",
        "type": "string" 
    }]
}
```

- **meta.json**: Special definitions used by Dexter are stored here.
    * **title (string, required)**: The user-friendly title of your module
    * **icon (string, optional)**: A [FontAwesome icon](http://fortawesome.github.io/Font-Awesome/cheatsheet/) that will show up alongside your module in the App editor.
    * **inputs (array, optional)**: Definitions for any inputs your module accepts. Each item in the array should have 3 properties:
        * *id (string)*: The input's code-friendly key (ex: first_name)
        * *title (string)*: The input's user-friendly name (ex: First Name)
        * *type (string)*: A basic type (string, integer, array)
    * **outputs (array, optional)**: Definitions for any outputs your module generates. Just like inputs, each object in the array has 3 properties:
        * *id (string)*: The input's code-friendly key (ex: first_name)
        * *title (string)*: The input's user-friendly name (ex: First Name)
        * *type (string)*: A basic type (string, integer, array)

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
            return this.succeed();
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

...once it gets inside the Dexter App, it ends up looking like:

`{ getData: fn, run: fn, log: fn, succeed: fn, fail: fn, log: fn }`

This implicit wrapping process lets you focus on the details of your module, and even run your module outside of Dexter without worrying about managing Dexter's dependencies!

<aside class="warning">
If your module exports properties named succeed, fail, log, or run, your module will not work!
</aside>

## Inputs
> Code

```javascript
module.exports = {
    dump: function() {
        console.log.apply(console, arguments);
        console.log('--------');
    },
    run: function(step) {
        var inputs = step.inputs(),
            singleExisting = step.input('singleExisting').first(),
            singleMissing = step.input('singleMissing').first(),
            singleMissingDefaulted = step.input('singleMissingDefaulted', 'singleDefault').first(),
            
            multipleExisting = step.input('multipleExisting').toArray(),
            multipleMissing = step.input('multipleMissing').toArray(),
            multipleMissingDefaulted = step.input('multipleMissingDefaulted', 'multipleDefault').toArray(),
            
            multipleExistingZero = step.input('multipleExisting')[0],
            multipleExistingFirst = step.input('multipleExisting').first(),
            multipleMissingZero = step.input('multipleMissing')[0],
            multipleMissingFirst = step.input('multipleMissing').first();
            
        this.dump(inputs);
        this.dump(singleExisting, singleMissing, singleMissingDefaulted);
        this.dump(multipleExisting, multipleMissing, multipleMissingDefaulted);
        this.dump(multipleExistingZero, multipleExistingFirst, multipleMissingZero, multipleMissingFirst);
    }
};
```

> Result

```javascript
{ singleExisting: 'Foo', multipleExisting: ['Bar', 'Baz'] }
'-------'
{ 'Foo', null, 'singleDefault' }
'-------'
{ ['Bar', 'Baz'], [], ['multipleDefault'] }
'-------'
{ 'Bar', 'Bar', undefined, null }
```

The real power of Dexter lies in its ability to mix and match outputs from a variety of sources in order to provide input data for a given module. Well designed modules must therefore be able to accommodate as broad a spectrum of inputs as possible.

All module inputs are, under the hood, wrapped up in a single object `{}`. The input object's properties are explicitly defined in the module's meta.json file, and each defined property is always present in the compiled input `{ input1: null, input2: null}`. When an App is configured, its creator maps different data sources to each of your module's inputs `input1 = output1, input2 = output2, {output 1: 1}, {output 2: 2}`. When the app is executed, those mappings are evaluated, and the results bound to the input object `{input1: 1, input2: 2}`.

The real fun begins when input data is mapped from sources that have multiple outputs `[{output1: 1}, {output1: 2}], {output2: 3}`.  Dexter's input system accommodates data sources with multiple outputs, so all available outputs get aggregated into our input object `{ input1: [1,2], input2: [3] }`. It's also possible that some configured mappings will produce no data at all `{ input1: null, input2: 3 }`.

It's your job as a module creator to figure good rules for dealing with this variety of possible input data. Is a particular input critical enough to warrant killing an App if it's missing?  How do you deal with misaligned amounts of input?  Which inputs can you only deal with 1 of, and which are safe to handle as arrays?

Dexter provides some tools to help you execute your decisions.  Inputs can either be accessed directly by extracting them via `step.inputs()`, or by wrapping individual inputs in a Dexter [data collection](#data-collections) by calling `step.input(key)`. You can then verify that you have the data you need through any combination of assertions (assertion errors will cause the app to abort), explicit failures (calling `this.fail(...)` lets you give specific reasons for the error), or by providing sane defaults (calls to `input(key, default)` with a good fallback default value).

## Outputs
> Module 1: User fetcher

```javascript
module.exports = {
    run: function() {
        this.succeed({ email: 'joe@example.com', name: 'Joe' });
    }
};
```

> Module 2: Link fetcher

```javascript
module.exports = {
    run: function() {
        this.succeed([
            { url: 'http://rundexter.com', title: 'Dexter' },
            { url: 'http://reddit.com', title: 'Why my project is late' }
        ]);
    }
};
```

> Module 3: Message

```javascript
//Imports are:
// to = dexter.step('module_1').output('email');
// urls = dexter.step('module_2').output('url');
module.exports = {
    run: function(step) {
        var to = step.input('to'),
            urls = step.input('url');
        console.log(to.toArray(), urls.toArray());
        return this.succeed();
    }
};
//In the log:
// ['joe@example.com'] ['http://rundexter.com', 'http://reddit.com']
```    

In keeping with the theme of doing one thing and doing it well, Dexter assumes all modules will output a single kind of object that reflects its purpose. Does the module get Slack history?  Then the output should look like `{ user, text, link }`. Does it get a list of stock prices?  The output should look like `{ symbol, name, price, change }`. You describe what that object looks like in your `meta.json` file, then ensure that those properties exist in the object(s) you export via `this.complete()`.

That doesn't mean you're restricted to returning a single object. If your module didn't produce any output, just return nothing. If it produced multiple data points, return a collection of your output objects. So long as each object has properties that are recognizable from your `meta.json:outputs` definitions, Dexter will be able to parse the outputs and pass them along as needed.

What you *don't* want to do is to define and return complex object properties. Dexter's power lies in the ability to map the output from one module to the input of another, and to let the App developer decide how to make those associations meaningful. In order for you output to be understood by the widest variety of other modules, each object property should be both of a basic type (string/numeric good, binary/object bad) and well named (email/url good, o_fsk_n50/aws_9009939923f_useast1 bad).

That doesn't mean you *can't* output complex objects. For instance, if you're building a suite modules that perform image manipulations, you *should* have a standard base64-encoded image binary type and support it throughout the suite. Or, if you're doing an AWS automation App, you should have IAM objects as a basic type that your modules understand.

You can read more about outputs under [BaseModule::succeed](#basemodule-succeed-data).


## Data collections

Calls to `step.input` and `step.output` return data wrapped up by a Dexter collection, which makes managing an uncertain amount of data easier.

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

### BaseModule.succeed(data)
> No output

```javascript
return this.succeed();
```

> Single output

```javascript
return this.succeed({ foo: 'Foo', bar: 'Bar' });
```

> Multiple outputs

```javascript
return this.succeed([
    { foo: 'Foo', bar: 'Bar' },
    { foo: 'Baz', bar: null }
]);
```

Parameter|Type|Description
---------|--------|-----------
data | array,object | An object containing your outputs, or an array of objects containing your outputs if necessary

When your module has successfully finished its job, it needs to call `succeed()` to let Dexter know it's safe to continue the current App.

If you have any useful information you want to make available to other modules, you'll want to return it here. Dexter assumes that you're returning an object of some kind, and that the object only has properties that match those you've defined in your `[meta.json:output](#anatomy-of-a-module)`. Your app can also generate multiple outputs by returning an array of such objects - as long as each object's properties are correctly configured, Dexter will still know how get and share the data with other modules.

<aside class="warning">
Note that you should only call succeed once, and after you do, the wrapper running the function will quickly shut down, making any code your run afterwards a bit of a gamble. Calling `return this.succeed()` as shown in the code samples is one way to guarantee that you only make one call and don't cause a race condition afterwards, though you can use careful code flow to do the same if you'd prefer.
</aside>

<aside class="success">
<p>Your response object properties can technically be *anything* - strings, arrays, objects - pretty much any non-binary, non-stream, non-executable, serializable value is fair game!</p>
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
Note that you should only call `fail()` once, and after you do, the wrapper running the function will quickly shut down, making any code your run afterwards a bit of a gamble. Calling `return this.fail()` as shown in the code samples is one way to guarantee that you only make one call and don't cause a race condition afterwards, though you can use careful code flow to do the same if you'd prefer.
</aside>

### BaseModule.log(details)
> Simple log

```javascript
this.log('Querying Foo...');
```

> Detailed log

```javascript
this.log({ message: 'Back from foo', foo: foo.results() });
```

> Formatted log

```javascript
this.log({ message: 'Found foo :foo and bar :bar', foo: foo.results(), bar: 42 });
//Shows up in the console as:
// 'Found foo **Object(...)** and bar 42'
//where the foo object is an interactive console object and 42 is a colored number
```

A logger is a programmer's best friend, and the Dexter logger gives you a  lot of power. Every time you write a log, Dexter saves all the explicit data you provide alongside a snapshot of the App at that particular point in time. You can then recall and play back your logs for a particular run of an App and watch the App's context progress as the logs come in.

In its most basic form, you can pass a simple `msg` string into the logger. Note that this is the exact same as calling `log({ message: msg })`.

If you've got some specific information you want to keep track of, you can pass it along as well. Our log playback tools will give some special treatment to any `message` property you send along (it'll show it in front of the rest of the logged data in the Dexter console), but it's optional if you don't want it.  Remember that you can an entire copy of the data in the `dexter` object for free - there's no need to duplicate its information in your log data.

The `message` property gets additional special treatment by allowing you to format your log message in a way that takes advantage of the modern browser console. You can actually embed specific data in your log message by referencing its key after a colon, allowing you to easily see that data in the browser console. This is useful if you want to be able to monitor specific logged variables at a glance in the console.

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
    this.succeed();
} else {
    dexter.setGlobal('lastIndex', loopTo);
    this.replay();
}
```

One of the downsides of working with APIs is that many of them are rate limited. This poses a problem if you're trying to handle an arbitrary amount of data in the 60s time window allotted to a Dexter module.  If you're in a situation where you're not sure if you can handle all the data you've been given, `replay()` can help you out.

Calling replay will rerun the current module with the exact same inputs it was originally given. By using the App's global state management tools, your module can keep running until it finishes what it set out to do.

Some other scenarios where replay might be useful:
* Waiting for an API endpoint to become available
* Waiting for slowly updated data to change
* Waiting for scarce resources to free up

<aside class="warning">
There is currently a 50-step limit built into Dexter. That's more than enough for most Apps, *unless* you're relying heavily on replay to accomplish your goals. If you think you'll need to replay enough times to worry about that limit, you might want to consider another strategy, like queue and fetch combined with timers.
</aside>

## App data
### dexter
This is the 2nd parameter passed into your module's `run()` function, and represents a high-level view into all that's gone on in your app so far.

It's also the accessor you use to map inputs and add switch cases when wiring up your App - more on that in the App UI section!

### dexter.clone()
```javascript
var d2 = dexter.clone();
d2.setGlobal('test', 1);
assert.notEqual(d2.global('test'), dexter.global('test'));
```
Make a deep copy of the entire Dexter structure. This is mainly useful if you need to pass the structure off to a subprocess and need to ensure that it doesn't make any changes.

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

Right now we're in the process of giving you as a module developer a way to communicate to App developers which keys your module requires. For now, just test for the keys' presence and return an informative message indicating the requirement.

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

### dexter.url(key, default)
```javascript
var dexterUrl = dexter.url('home');
```

URLs used to access internal Dexter resources. These are better used through the actual helper functions and modules rather than directly by your modules.

Key|Type|Description
---------|--------|-----------
home | string | Dexter's homepage
logger | string | An endpoint used to track in-App logs
reporter | string | If this workflow is being run on a timer, the App calls back to the reporter to let it know it's done.


### dexter.global(key, default)
```javascript
var cacheKey = step.module('name') + '_cache',
    cache = dexter.global(cacheKey, null);
if(!cache) {
    cache = this.calculate();
    dexter.setGlobal(cacheKey, cache);
}
this.succeed({ data: cache });
```

Fetch globally stored information assigned during the use of this app. We're not placing any restrictions on how you use this, so you should do so both carefully and sparingly.

Available keys and values are determined by individual modules and will vary from App to App.

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
this.succeed({ data: cache });
```

An App-wide data store for the entire instance of the app. This is the ONLY place you can store information explicitly between steps and modules. We're not placing any restrictions on how you use this, so you should do so both carefully and sparingly.

Any serializable data can be assigned to a global (string, numeric, object, array, etc.). Unless you have a good reason not to, it's a good practice to namespace your global variables for your particular module. 

### dexter.step(id)
```javascript
var linkedStep = dexter.step('linked-step', null),
    data = step.input('data', (linkedStep) ? linkedStep.input('data') : null);
return this.complete({ data: this.process(data) });
```

Get a specific step from the workflow. There shouldn't be any real reason to use this, but if you do, it outputs either an `[AppStep](#step)` or `null`;
    

## Step

This is the first parameter passed to your module's `run()` function, and probably the most important. It contains both the configuration of your module for the current App as well as the data that's been assigned to it.

### step.clone()
```javascript
var stepClone = step.clone();
```

Make a deep copy of the entire step.

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

<aside class="warning">
Note that for trigger steps, these values will always return the default, as triggers have no modules.
</aside>

### step.input(key, default)
```javascript
var email = step.input('email', 'default@example.com').first(),
    messages = step.input('message'),
    message = messages.toArray().join('<br>');
this.queueEmail(email, 'You have messages', message);
return this.succeed();
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

### step.output(key, default)
```javascript
var output = step.trigger().output('url');
console.log(output.length, output.toArray())
//outputs: [{ url: 'http://google.com' }, { url: 'http://yahoo.com' }]
//log: 2 ['http://google.com', 'http://yahoo.com']
```

Extract a single property's worth of output data from a step. This isn't often used inside of a module, but *is* often used inside a switch statement to compare information. Like inputs, this returns a [Dexter collection](#data-collections)

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