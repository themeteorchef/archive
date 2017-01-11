In this snippet, we'll learn how to make use of [NPM packages](http://npmjs.org) in our Meteor application. We'll learn how to do it manually using Meteor packages, as well as how to speed the process up by using the `meteorhacks:npm` package to automate everything for us.

Something that's interesting but rarely mentioned is that when we're writing Meteor applications, we're technically writing [Node.js](nodejs.org) applications. Although Meteor gives us a nice JavaScript API for writing client and server code, under the hood, Node.js is running the show. What's neat about this is that many of the resources available to developers writing Node applications are also available to Meteor developers. One of those resources—packages from the [NPM package repository](http://npmjs.org)—can be included in Meteor applications with little effort, adding over 187,000 packages to the existing 7,500 we can find on [Atmosphere](https://atmospherejs.com) today.

What's great about this is that if a Meteor package doesn't exist for a problem you're trying to solve, chances are an NPM package _does_ exist. In this snippet, we're going to look at how we can take advantage of this and get access to NPM packages in our Meteor application.

## NPM packages vs. Meteor packages
Before we look at how to implement NPM packages, it's important to understand how they differ from Meteor packages. First, while Meteor packages live on Atmosphere, NPM packages live on...[NPM](http://npmjs.org)! Both fulfill the same exact purpose: hosting a remote repository of publicly available code that developers can add to their applications.

An NPM package is denoted by a [Node module](https://docs.npmjs.com/getting-started/creating-node-modules), combined with a [package.json](https://docs.npmjs.com/files/package.json) file in a directory. For example, the following two blocks of code could identify a simple NPM package.

<p class="block-header">index.js</p>

```javascript
exports.eatSandwich = () => {
  console.log( "Oh yeah, now we're eating sandwiches!" ); 
}
```

<p class="block-header">package.json</p>

```javascript
{
  "name": "eat-sandwich",
  "version": "1.0.0",
  "description": "You call yourself a sandwich eater?",
  "author": "Doug Funny <doug.funny@bluffington.edu>",
  "license": "MIT"
}
```

With these two files, we'd technically have an NPM package. Notice that there's nothing terribly special about this. We have some JavaScript code and a file to describe that JavaScript code. Cool! On the Meteor side, we have a few more things going on, but more-or-less the same concept:

<p class="block-header">eat-sandwich.js</p>

```javascript
eatSandwich = () => {
  console.log( "Oh yeah, now we're eating sandwiches!" ); 
}
```

<p class="block-header">package.js</p>

```javascript
Package.describe({
  name: "themeteorchef:eat-sandwich",
  summary: "You call yourself a sandwich eater?",
  version: "1.0.0",
  git: "https://github.com/themeteorchef/eat-sandwich",
  documentation: "README.md"
});

Package.onUse( function( api ) {
  api.versionsFrom( '1.2.0.1' );
  api.addFiles( [ "eat-sandwich.js" ], [ 'client', 'server' ] );
  api.export( "eatSandwich", [ 'client', 'server' ] );
});
```

A little bit different. Notice that with Meteor packages, we use a `JavaScript` file as opposed to a `JSON` file to describe our package. Both of these packages achieve the exact same thing: when we call `eatSandwich()` in our application, we'll see `"Oh yeah, now we're eating sandwiches!"` printed to the console. Again, we use different conventions to define our packages, but at the end of the day we're just writing JavaScript. Make sense?

<div class="note success">
  <h3>Writing your own packages <i class="fa fa-thumbs-up"></i></h3>
  <p>Curious about how packages work in Meteor and want to learn how to write your own? Check out the <a target="_blank" href="http://themeteorchef.com/recipes/writing-a-package/">Writing a Package</a> recipe!</p>
</div>

## Adding an NPM package with a Meteor package
How meta are we?! One way that we can add NPM packages to our Meteor application is by...using a Meteor package! 
 
<blockquote width="100%" class="twitter-tweet" lang="en"><p lang="en" dir="ltr">&quot;What&#39;s bower?&quot;&#10;&quot;A package manager, install it with npm.&quot;&#10;&quot;What&#39;s npm?&quot;&#10;&quot;A package manager, you can install it with brew&quot;&#10;&quot;What&#39;s brew?&quot;&#10;...</p>&mdash; Stefan Baumgartner (@ddprrt) <a href="https://twitter.com/ddprrt/status/529909875347030016">November 5, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

As part of the Package API in Meteor, we get access to a method called [Npm.depends](http://docs.meteor.com/#/full/Npm-depends) that allows us to specify NPM packages that our Meteor package depends on. Although this method is designed to load up NPM packages for a _Meteor package_, we can technically piggyback on this functionality and use it to load NPM packages into our application. Here's how it works:

<p class="block-header">Terminal</p>

```javascript
meteor create --package npm-packages
```

First, we [create a new package](http://themeteorchef.com/recipes/writing-a-package/) from within our Meteor project. Next, we modify the `package.js` file within this new package to include our NPM packages and a file where we'll handle the actual loading of the package.

<p class="block-header">/app/packages/npm-packages/package.js</p>

```javascript
Package.describe({
  name: 'themeteorchef:npm-packages',
  version: '0.0.1',
  summary: 'Load NPM packages into our Meteor application.'
});

Npm.depends({
  "lodash": "3.10.1"
});

Package.onUse( function( api ) {
  api.addFiles( 'npm-packages.js', [ 'server' ] );
});
```

This bears some explaining. The part to pay attention to is the `Npm.depends()` block. Here, we're passing in an object which contains a list of NPM packages that we want to depend on in our application (for our example, we want to load the [lodash JavaScript utility library](https://lodash.com/)). The keyword here is _depend_. At this point, we're only telling Meteor to create a link between our Meteor package (here, `themeteorchef:npm-packages`) and the NPM packages we specify in the `Npm.depends()` block. We are _not_ loading these packages yet, and they're not available to use. To make that happen, we need to modify the `npm-packages.js` file we're including on the server.

<div class="note">
<h3>NPM packages only available on the server <i class="fa fa-warning"></i> </h3>
<p>As of writing, we can only load NPM packages using this method on the <em>server</em>. This is due to the nature of how Node works as a server-side environment.
</div>

<p class="block-header">/app/packages/npm-packages/npm-packages.js</p>

```javascript
Meteor.npmPackage = function( npmPackage ) {
  var package = Npm.require( npmPackage );
  return package;
};
```
Inside of our `npm-packages.js` file, we've defined a new function on the `Meteor` namespace `.npmPackage()`. This function is designed to act as a shortcut for actually loading our NPM packages. Remember, at this point, `Npm.depends()` has only made our package _aware_ of the NPM packages we're interested in, but it hasn't made them _available_ to our application. To do that, we need to call `Npm.require()` passing the name of the package we want to load. 

In our example, then, we'd call `Npm.require( 'lodash' )` to get access to the `lodash` package we specified in `Npm.depends()`. Notice that here, we're assigning our `Npm.require()` to a variable (accepting a variable that was passed when `Meteor.npmPackage` was called) and then returning that from our function. Why? This allows us to load _any_ NPM package we add using `Npm.depends()` as opposed to just one at a time. To make sense of that, here is what this would look like _without_ the `Meteor.npmPackage` function:

<p class="block-header">/app/packages/npm-packages/npm-packages.js</p>

```javascript
Lodash = Npm.require( 'lodash' );
```

<p class="block-header">/app/packages/npm-packages/package.js</p>

```javascript
Package.describe({
  name: 'themeteorchef:npm-packages',
  version: '0.0.1',
  summary: 'Load NPM packages into our Meteor application.'
});

Npm.depends({
  "lodash": "3.10.1"
});

Package.onUse( function( api ) {
  api.addFiles( 'npm-packages.js', [ 'server' ] );
  api.export( 'Lodash', [ 'server' ] );
});
```

Spot the difference? Here, we add a step. Before, we were assigning a new function to the `Meteor` namespace which is automatically accessible in our application. In this new version, we're loading just the single package `lodash` by assigning it to a global variable `Lodash`. To make that actually accessible in our application, then, back in our `package.js` file we need to add a call to `api.export( 'Lodash' )` so that Meteor knows to export the global variable we defined. So...what's the real difference?

### Loading NPM packages in our application
In our application code—not our package code—we now have access to the NPM package that we loaded. To make use of it, though, we have to require it one more time in the file where we'd like to use it. Using our more recent global variable pattern:

<p class="block-header">/app/server/use-lodash.js</p>

```javascript
var _ = Lodash;

console.log( _.chunk( [ 1, 2, 3, 4 ], 2 ) );
```

Notice that here, we already have access to the global `Lodash` variable we exported from our package. For convenience, we assign the global variable to a local one `_` (this is a naming convention taken from the lodash library, not a typo or convention specific to this snippet). From there, we have access to the code in the `lodash` package we loaded from NPM! As an example, we attempt to call the [chunk](https://lodash.com/docs#chunk) method from the library and log out the results to our console.

![Result of calling lodash's chunk method and logging it to the server console.](http://cl.ly/image/1u3s0s1c0l3Q/Image%202015-09-24%20at%2010.03.51%20PM.png)

Cool! As expected, we get back our array of four numbers broken up into chunks of two. 

So...this worked okay, but let's update this code to use our first example where we defined a function called `Meteor.npmPackage`:

<p class="block-header">/app/server/use-lodash.js</p>

```javascript
var _ = Meteor.npmPackage( 'lodash' );

console.log( _.chunk( [ 1, 2, 3, 4 ], 2 ) );
```

Hmm...this doesn't seem very different? On the contrary, friend! Remember that this function `Meteor.npmPackage` is automating the process of calling `Npm.require()` for us from within our package. What this translates to is removing the need for defining and exporting global variables in our package code _every single time_ we want to add a new NPM package. Instead, we can just update our package to require a new dependency:

<p class="block-header">/app/packages/npm-packages/package.js</p>

```javascript
[...]

Npm.depends({
  "lodash": "3.10.1",
  "chalk": "1.1.1"
});

[...]
```

And then, back in our application code....

<p class="block-header">/app/server/use-lodash.js</p>

```javascript
var _     = Meteor.npmPackage( 'lodash' ),
    chalk = Meteor.npmPackage( 'chalk' );

chalk.enabled = true;

console.log( _.chunk( [ 1, 2, 3, 4 ], 2 ) );

console.log( chalk.green( "Holy cow!" ) );
console.log( chalk.yellow( "Are you serious?!" ) );
console.log( chalk.red( "This is awesome!" ) );
```

![Printing chunked array and coloring our console.logs like champions!](http://cl.ly/image/2B1P463l460i/Image%202015-09-24%20at%2010.19.29%20PM.png)

Oh yeah! Look at that. Go ahead, get out your pen. Write a letter home. Let them know that the next time they see you, you won't be yourself because _your mind has been blown_.

While this is cool, it _does_ still require us to open up our package code and make edits every time we want to add a new NPM package. If we're only using one or two this may be okay, but what if we're NPM _fiends_?

## Adding an NPM package with `meteorhacks:npm`
If we want to skip all of the hullabaloo with adding our own package just to add packages, our dear ol' pal [Arunoda](http://twitter.com/arunoda) came up with a slightly better way. The irony? You have to add a package. Go ahead, dry your eyes. Here's how to get it working:

<p class="block-header">Terminal</p>

```bash
meteor add meteorhacks:npm
```

When you do this, the package will do two things: create a new package within your application called `npm-container` and then add a file called `packages.json` to the root of your project. When you first add `meteorhacks:npm`, you may find that your application stops, printing a message for you to restart it. This is expected and allows `meteorhacks:npm` to get set up. Once this is done, to add NPM packages to our application all we need to do is update our `packages.json` file that was created for us:

<p class="block-header">packages.json</p>

```javascript
{
  "lodash": "3.10.1",
  "chalk": "1.1.1"
}
```

<p class="block-header">/app/server/use-lodash.js</p>

```javascript
var _     = Meteor.npmRequire( 'lodash' ),
    chalk = Meteor.npmRequire( 'chalk' );

chalk.enabled = true;

console.log( _.chunk( [ 1, 2, 3, 4 ], 2 ) );

console.log( chalk.green( "Sri Lankan simplicity: 1" ) );
console.log( chalk.red( "Jumping through hoops: 0" ) );
console.log( chalk.white( "-----------------------" ) );
console.log( chalk.yellow( "Thanks, Arunoda!" ) );
```
![Example output getting the same result with less steps](http://cl.ly/image/2Y3i32431b3y/Image%202015-09-24%20at%2010.40.40%20PM.png)

Sweet! Pay close attention to our variable declarations. It's subtle, but instead of using the `Meteor.npmPackage()` method that we defined earlier, we swap this with a method given to us by the `meteorhacks:npm` package called `Meteor.npmRequire()`. The punchline? This is doing the exact same thing as our own function behind the scenes, but automating the process of updating our package list by letting us use a `JSON` file instead. Nice!

## Takeaways
- NPM packages are just like Meteor packages in respect to what they accomplish, but rely on different APIs and organizational patterns to get the job done.
- We can get access to NPM packages by adding our own package, or, automate the process using `meteorhacks:npm`. 
- Using NPM packages is something every Meteor developer should know how to do as it only serves to enhance your work.
