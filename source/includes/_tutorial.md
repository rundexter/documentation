# Tutorial

## Goal
Let's look at how to create a non-trivial module for Dexter.  The goal of this module will be to consume an RSS feed and to apply an optional text filter against the content.  In the process, we'll take advantage of some 3rd party tools, figure out how to deal with variable inputs, and learn how to deal with errors.  Let's get started!

## Phase 0: Preparation
Before we get started, you should make sure the following things are true:

1. You've signed up at http://rundexter.com
1. You've installed the [sdk](#sdk-overview)
1. You've successfully added a key to your environment
1. You've got a *basic* understanding of [Node.js](http://nodeguide.com/beginner.html)
1. You're ready to build cool things!

## Phase 1: Setup
```shell
dexter create "`whoami`: RSS Filter" \
    && cd `whoami`_rss_filter
```
To begin, we'll use the Dexter SDK to create a module, giving its title a namespace (prefix) to distinguish it from other people who are also following this tutorial.  The sample command will use your computer's username as the namespace, but feel free to replace ``whois`` with another name.

If someone else has used that module name before, you'll get a warning.  Just change the title and try again until you find a unique name!

At this point we have a module skeleton, our package name (`package.json:name`) and module title (`meta.json:title`) already set, git initialized and ready to push to Dexter, and the foundation for our tests ready to go.  Pretty good for a single command!

<div></div>
```shell
$ npm install --save lodash
$ npm install --save request
$ npm install --save feedparser
```

The main thrust of this module is to deal with RSS feeds, so we'll want to add a [lovely package](https://github.com/danmactough/node-feedparser) that elegantly handles a variety of feed types.  Feedparser doesn't actually fetch the feed XML, however, so we'll also need a package to deal with [remote requests](https://github.com/request/request).

We'll also use [lodash](https://lodash.com/) to make our code a bit more readable.

Since the module is just a Node library on steroids, we can use `npm install --save` to add our dependencies directly to our `package.json`.  Handy!

<div></div>

```javascript
module.exports = {
    run: function(step) {
    }
}
```

The last bit of preparation is to get rid of the sample code in `run()` that comes with the skeleton.  We'll replace it with something better soon!


## Phase 2: Remote requests
```javascript
var _ = require('lodash'),
    request = require('request');
module.exports = {
    //...run
    fetchUrl: function(url, callback) {
        var req = request(url);
        req.on('error', callback);
        req.on('response', function(resp) {
            if(resp.statusCode != 200) {
                return callback(new Error(
                    'Bad status code ' + resp.statusCode + ' for ' + url
                ));
            }
            return callback(null, this);
        });
    }
}
```

Before we can start processing feed data, we've got to go get the feed itself.  To do that, we're going to throw together a quick implementation of the Request library.

The process is simple...if we see an error, we fire it straight back through our callback.  If we get a response, we make sure it's `200 OK` (again erroring out if it's not), then pass the raw response along to our callback.

Note that nothing we've done so far is Dexter-specific - this is just plain, standard Node.js code so far.

## Phase 3: Feed parsing
```javascript
var request = require('request'),
    FeedParser = require('feedparser');
module.exports = {
    //...run
    //...fetchUrl
    fetchItems: function(stream, callback) {
        var parser = new FeedParser()
            , items = [];
        parser.on('error', callback);
        parser.on('end', function(err) {
            if(err) {
                return callback(err);
            }
            return callback(null, items);
        });
        parser.on('readable', function() {
            var item;
            while((item = this.read())) {
                items.push(item);
            }
        });
        stream.pipe(parser);
    },
}
```

Now that we know how we're getting our http data, we now need to push it through FeedParser.  FeedParser deals entirely in streams - you take any stream data (specifically any stream data that's supposed to contain an RSS file) and pass it into your FeedParser instance.  The actual processing of content happens in events it emits.

Our use of these events is pretty straightforward.  Did we see an error? Pass it up through the callback.  Have we streamed enough readable data to act on?  Stash the data we found in an `items[]` array.  Are we at the end of the stream?  We're done, call back with the stuff we found.

Again, we've done nothing special or Dexter-y yet.  We've added a few 3rd party libraries to a Node.js library and wrapped up their basic functionality with some callbacks.  Let's get to the more interesting parts!

## Phase 4: Wiring it up
```javascript
//...requires
module.exports = {
    run: function(step) {
        var url = step.input('url').first(),
            self = this;
        if(!url) {
            this.log('No url provided, returing to App');
            //It's not really a critical failure to not have an 
            //  url to work with, so we'll roll on.
            return this.complete();
        }

    },
    //...fetchUrl
    //...fetchItems
};
```

First things first - let's deal with our inputs.  For now we're just going to focus on a single input: the feed URL.  To keep things simple, our module is just going to deal with a *single* RSS feed - dealing with loops will be a topic for another day. (hint: it means calling toArray() instead of first().  Spoiler alert!)

So what are the possibilities for our input?  Ideally we just have one input - that's the only real use case we're set up for.  However, we could have multiple inputs, but since we're using the first() function, we're throwing everything but the first input away.

Now the only concern we have is weather or not we have *any* URLs to process.   What should we do if we have none?  Is it worth stopping the *whole* App just because we don't have anything to do?  Probably not.  In this case it makes the most sense to log the fact that we didn't get any data (just in case it makes things confusing elsewhere in the App) and move on as if the feed had no articles at all.

If you wanted to be *really* thorough, you could also:

* Validate the urls (it's probably worth throwing an error if the urls don't validate, since it means the App might be wired up wrong)
* Log a warning if we're throwing away URLs by using first()


<div></div>
```javascript
//...requires
module.exports = {
    run: function(step) {
        //...inputs
        this.fetchUrl(url, function(err, stream) {
            if(err) {
                return self.fail(err);
            }
            //Let the parser grab the data
            self.fetchItems(stream, function(err, items) {
                var response = [];
                if(err) {
                    return self.fail(err);
                }
                //Extract dexter-friendly data from the items
                _.each(items, function(item) {
                    response.push({
                        url: item.link,
                        title: item.title,
                        summary: item.summary,
                        author: item.author
                    });
                });
                return self.complete(response);
            });
        });

    },
    //...fetchUrl
    //...fetchItems
};
```

Finally, let's wire up our remote request and feed parsing functions to our Module tools.  We'll start with fetchUrl.  Right off the bat we *do* want to fail if an error occurred when connecting to the page - something broken usually isn't a good thing.  If we managed to extract a stream, we'll pass it along to fetchItems.

We'll follow the same rules for the result of fetchItem - if we hit an error when processing the page, we'll fail.  Otherwise, all that's left to do is to extract the basic properties we want our module to return and
send them along to `complete()`.  Mission accomplished!

## Phase 5: Testing
> fixtures/default.js

```javascript
//...
data: {
    local_test_step: {
        input: {
            url: 'http://feeds.arstechnica.com/arstechnica/index/'
        }
    }
}
//...
```

Let's get ready to test.  For this module, our requirements are pretty simple - we just need to add some sample data to the step's inputs.  Our default test step is called `local_test_step`, the input we're targeting is `url`, and we like [Ars Technica](https://arstechnica.com) here at Dexter.  The code resulting from this logic is pretty straightforward and simple.

<aside class="success">
What you're looking at here in the fixture is the raw serialization data that drives a Dexter App.  When you're using the `step` and `dexter` objects inside your module, you're actually interacting with some friendly wrappers around this raw information.  While you should never work with this data directly in your modules, as the wrappers will hide any changes we might make to the underlying data structure, it can be useful to see what it looks like here.
</aside>

<div></div>
```shell
$ dexter run
...
```

If everything worked, you should see a list of the most recent stories on Ars Technica in your console!

## Phase 6: Preparing for deployment
> meta.json

```json
{
    "inputs": [
        {
            "id": "url",
            "title":"Feed URL",
            "type":"string"
        }
    ],  
    "outputs": [
        {
            "id": "url",
            "title":"Article URL",
            "type":"string"
        }, {
            "id": "title",
            "title":"Article title",
            "type":"string"
        }, {
            "id": "summary",
            "title":"Article summary",
            "type":"string"
        }, {
            "id": "author",
            "title":"Author",
            "type":"string"
        }
    ],  
    //...
}
```

The only special requirement a Dexter module has outside of run/complete/fail is the explicit input and output definitions in `meta.json`.  We need to identify each input we're expecting and define each output property we'll return.  This metadata is used to allow the Dexter App editor to wire up our module with other modules based on the user's requirements.

Our inputs and outputs are pretty straightforward.  We're just accepting a single input - an URL string - and outputting 4 different strings that we extract from the feed.

## Phase 7: Push version 0.0.1!
```shell
$ git commit -am 'First working version'
$ dexter push
```

That's it, you've got a working RSS parser module!  Just `dexter push`, and, once the process completes, you should be able to open the App editor and see your module.  Congrats, you're now a Dexter developer!

## Phase 8: Add an optional filter
```javascript
//..includes
module.exports = {
    run: function(step) {
        var url = step.input('url').first(),
            filters = step.input('filter'),
            self = this;
        //...
                if(filters.length) {
                    response = self.filterItems(response, filters);
                }
                return self.complete(response);
        //...
    },
    //..fetchUrl
    //..fetchItems
    filterItems: function(items, terms) {
        var response = [],
            lcTerms = []
        ;
        //We don't care about case sensitivity
        terms.each(function(term) {
            lcTerms.push(term.toLowerCase());
        });
        _.each(items, function(item) {
            //Check against ANY of the item's properties.
            _.forIn(item, function(val) {
                var match = false;
                //See if any of our terms hit
                _.each(lcTerms, function(term) {
                    if(val.toLowerCase().indexOf(lcTerm) >= 0) {
                        response.push(item);
                        match = true;
                        return false;
                    }
                });
                //Don't continue if we managed to match on a property
                if(match) return false;
            });
        });
        return response;
    }
}
```

Let's see what it takes to update a Dexter module.  We'll add a useful feature to our RSS parser that lets you optionally filter out articles that don't match at least one of a list of given terms.

The code is pretty straighforward.  We'll add a new input - filter - and get them all.  Then we'll add a new function that looks for the filters in a list of our response objects and only returns the ones that match.  Finally, if we even *have* a filter, we'll apply our filter function to our discovered items and only return matches.

Note that we're taking advantage of the Dexter collection object that carries our filter values.  It's got a built-in lodash _.each function that lets you easily iterate through its contents.  You also could call filters.toArray() to work with a native array if you'd prefer.

<div></div>
> fixture/default.js

```javascript
//...
data: {
    local_test_step: {
        input: {
            url: 'http://feeds.arstechnica.com/arstechnica/index/',
            filter: ['apple', 'samsung', 'htc']
        }
    }
}
```

Then we update our fixture with a filter variable, providing an array of data to make sure our list functionality works.  Go ahead and `dexter run` to make sure it works!  You might get empty data this time if no items have the word "apple" in it.

<div></div>
> meta.json

```json
{
    "inputs": [
        {
            "id": "filter",
            "title":"Text filter",
            "type":"string"
        }
        //...
}
```

We'll also want to add our new input to the meta.json so it shows up in the App editor.

<div></div>
> package.json

```json
{
    //...
    version: '0.1.0',
    //...
}
```

Finally, in the only really new step for the update, we'll bump the version by a minor step since we added functionality.

That's it!  Just `git commit -am 'Added a filter'`, `dexter push`, and we now have a filterable RSS feed parser!
