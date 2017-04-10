In this snippet we'll learn about different ways we can organize our Meteor projects and some of the rules we have to follow. We'll do a deep dive on how projects at The Meteor Chef are organized and then pull in some other examples from the community as well. 

One of the first things a lot of people ask when getting started with Meteor is "how do I organize my project?" This is one of those more difficult features of Meteor to wrap your head around. Where other frameworks and platforms enforce a structure, Meteor does not. It _does_ offer a small set of conventions in respect to naming directories, but for the most part, how your project is organized is up to you.

Because of this, there's a lot of fear around whether or not we're "doing it correctly." The reality is that there _isn't_—yet, at least—a 100% correct way to organize your Meteor project. The Meteor Development Group (MDG) have made a nod to offering up a suggested structure in the future, but details of how this will work are fuzzy. In light of this, we're going to look at a few different ways to structure our projects, discuss the upsides and downsides of each method, and try to understand how files and directories behave within Meteor.

## Special Directories
In a Meteor project, there are just a few conventions when it comes to where our code _should_ live. Meteor refers to the directories in the list below as "[special directories](https://docs.meteor.com/#/full/structuringyourapp)."

<table>
 <thead>
   <tr>
     <th width="35%">Directory</th>
     <th>Description</th>
   </tr>
 </thead>
 <tbody>
   <tr>
     <td><code>/client</code></td>
     <td>From the root of our project, the <code>/client</code> directory is used to store all code that's meant to run on the client-side of our application. Any files located in this directory will be loaded on the <em>client-side</em> (browser) only. This is used as an alternative to writing our code in <code>if ( Meteor.isClient ) {}</code> blocks.</td>
   </tr>
   <tr>
     <td><code>/server</code></td>
     <td>From the root of our project, the <code>/server</code> directory is used to store all code that's meant to run on the server-side of our application. Any files located in this directory will be loaded on the <em>server-side</em> only. This is used as an alternative to writing our code in <code>if ( Meteor.isServer ) {}</code> blocks.</td>
   </tr>
   <tr>
     <td><code>/public</code></td>
     <td>From the root of our project, the <code>/public</code> directory is used to store all <em>files</em> that are meant to be served publicly. Images, graphics, and other static assets can live here. Note: these will live directly off the root URL of your application like <code>http://localhost:3000/file.jpg</code>.</td>
   </tr>
   <tr>
     <td><code>/private</code></td>
     <td>From the root of our project, the <code>/private</code> directory is used to store private data files and can only be accessed on the server. Files in this directory can be loaded on the server using the <a href="http://docs.meteor.com/#/full/assets">Assets API</a>. For example, things like email templates or seed data can be stored here.</td>
   </tr>
   <tr>
     <td><code>/client/compatibility</code></td>
     <td>From your <code>/client</code> directory, the <code>/compatibility</code> directory is for "JavaScript libraries that rely on variables declared with var at the top level being exported as globals. Files in this directory are executed without being wrapped in a new variable scope. These files are executed before other client-side JavaScript files."</td>
   </tr>
   <tr>
     <td><code>/tests</code></td>
     <td>From the root of our project, the <code>/tests</code> directory is not loaded anywhere and is intended for storing test code (e.g. see the <a href="https://velocity.readme.io/">Velocity framework</a>).</td>
   </tr>
 </tbody>
</table>

Aside from these directories, the sky is the limit. One directory that's _not_ mentioned in this list but deserves attention is the `/lib` or `/both` directory. From the root of our project, the `/lib` directory is used to store all code that's meant to run on _both_ the client and server. For example, storing Meteor methods in this folder will allow them to run in simulation (as stubs) on the client and for real on the server.

<figure>
  <img src="https://cl.ly/image/1w0M0m0z3q0I/both-cs-example.gif" alt="Example of running code on client and server.">
  <figcaption>Example of running code on client and server.</figcaption>
</figure>

**Note: technically, any directory stored in the root of the project with a name other than those in this table will make code accessible on both the client and server**. For example, `/tacos` will behave identical to `/lib` or `/both` in the root.

## Load order
One of the more unique features of Meteor is its ability to automatically load files in your project. Whenever you add, remove, or update a file: Meteor will handle the change. If you've ever worked with something like Live Reload, this is like that on steroids. In order to facilitate all of this, Meteor follows a set of five rules to determine when files are loaded into the application. From the [Meteor Documentation](https://docs.meteor.com/#/full/structuringyourapp):

1. HTML template files are always loaded before everything else
2. Files inside any lib/ directory are loaded next
3. Files with deeper paths are loaded next
4. Files are then loaded in alphabetical order of the entire path
5. Files beginning with main. are loaded last

Phew! This can be a lot to think about. The majority of the time you shouldn't have to worry about this, but it's important to keep it in mind. If you find yourself defining code in your application but keep getting `undefined` errors, it's likely that the issue can be attributed to load order. If you absolutely _must_ get around load order, your best options are to store your code within a package where you can explicitly control load order, or, wrap your code in a [`Meteor.startup()`](https://docs.meteor.com/#/full/meteor_startup) block (this will run your code after everything else has loaded).

## Organization patterns
Now that we have an understanding for Meteor's special directories and loading preferences, let's take a look at a few different examples for how we can organize our project. Keep in mind, none of these are The Right Way™. Each of these work just as well as the others. Find the one that makes the most sense to you and use that. Even better, make tweaks to these examples and invent your own! Let's start by looking at how things are organized at The Meteor Chef.

### The Meteor Chef
At The Meteor Chef, the name of the game in respect to organization is _clarity_. As much as possible, the goal is to use the clearest possible naming conventions when thinking about how directories and files are labeled. The point of this is to avoid a scenario where a developer browsing a TMC project has to ask "what is in here?" Of course, we can't mitigate confusion 100%, but we can get pretty close. As of writing (September 2015), here is the structure being used at TMC.

<div class="note">
  <h3>Internal Projects <i class="fa fa-warning"></i></h3>
  <p>This structure is mostly being used on internal projects right now and <em>not</em> public projects like <a href="https://github.com/themeteorchef/base">Base</a>. The structure below has only come into focus recently and hasn't been copied over to existing work. Leave that pitchfork by your side :)</p>
</div>

```coffeescript
/both
--- /methods
------ /insert
------ /update
------ /remove
------ /read
--- /modules
------ _modules.js
/client
--- /helpers
------ template.js
--- /lib
------ /vendor
--- /modules
------ _modules.js
--- /routes
------ authenticated.js
------ configuration.js
------ public.js
--- /stylesheets
--- /templates
------ /authenticated
--------- template-name.html
--------- template-name.js
------ /globals
--------- header.html
--------- header.js
------ /layouts
--------- application.html
------ /public
--------- template-name.html
--------- template-name.js
--- startup.js
/collections
/private
--- /email-templates
/public
/server
--- /modules
------ _modules.js
--- /publications
------ template-name.js
--- startup.js
/tests
application.html
package.json
settings-development.json
settings-production.json
```

Holy cow! There's a lot here, so let's step through it.

### /both
Again, any directory in the root of the project with a name other than those in the "Special Directories" list above is run on _both_ the client and the server. Although the convention here is generally to label this directory `/lib`, I've found `both` to be a little clearer about the intention of the directory and its contents. Inside of `both`, I store two directories: `methods` and `modules`. Methods should be pretty clear. This contains all of the Meteor methods that I write, broken up into sub-folders by type of operation: `insert`, `update`, `remove`, and `read`.

That last one, `read`, is optional. This is for methods that need to _read_ from the database and return a result to the client. I don't add this to every single project but it's good to note the convention.

Also in the `both` directory is a folder called `modules`. This is a newer convention. I've grown enamored with the [module pattern](http://toddmotto.com/mastering-the-module-pattern/) in JavaScript and have started incorporating it into all of my projects. The basic premise of the module pattern is that it encapsulates all of the code for a specific topic into a single file. More specifically, modules are handy for breaking complex tasks into singular functions. Let's look at how modules are working, along with an example of a module to make sense of how this all fits together.

<p class="block-header">/both/modules/_modules.js</p>

```javascript
Modules      = {};
Modules.both = {};
```

The first step is to create a namespace for our modules. Here, we define a global `Modules` assigning it to an empty object and then we define `Modules.both`, also set to an empty object. These are our namespaces. Notice that we're defining these inside of our `/both` directory, meaning `Modules` and `Modules.both` will be accessible on both the client and server.

<div class="note info">
  <h3>Also in /client and /server <i class="fa fa-info"></i></h3>
  <p>We'll breeze by it later, but we follow this same pattern on the client and server. We define <code>/client/modules/_modules.js</code> and <code>/server/modules/_modules.js</code>. Inside, we define what you'd expect <code>Modules.client = {};</code> and <code>Modules.server = {};</code>. Notice, we leave out <code>Modules = {};</code> here because we already have access to it from <code>/both/modules/_modules.js</code>!</p>
</div>

It's also important to pay attention to the file name. Notice that the file is called `_modules.js`. This isn't a typo. This ensures that this file gets loaded _first_ before all of the other files. Why? Because this contains the definition of the `Modules` variable that we'll need access to in other files (the `_` character puts the file first alphabetically and avoids the need for adding a `/lib` directory). This ensures that it's available when we need it.

Here's what defining a module looks like:

<p class="block-header">/both/modules/get-content.js</p>

```javascript
let collections = {
  recipe: Recipes,
  snippet: Snippets,
  blog: Blog
};

let setup = ( options ) => {
  var collection = _getCollection( options.type );
  return collection[ options.method ]( options.query || {}, options.projection || {} );
};

let _getCollection = ( type ) => {
  return collections[ type ];
};

Modules.both.getContent = setup;
```

This is a simpler module that's actually in use now in the app I use to manage The Meteor Chef, Kitchen. There are three distinct types of content in the app, each stored in their own collection. To account for this, I've defined a module that breaks _getting_ that content into two distinct steps. First, I define a function `setup` that gets returned publicly, aliased on the namespace `Modules.both.getContent`. This module is defined on the `both` namespace because it's used on both the client and server.

Inside, the module performs two steps. First, it grabs the corresponding collection based on the _type_ of content we pass when we invoke the function (we'll see this next). Once we have the collection, we run a query on that function, passing the query or projection defined when we invoke the function. It may be a bit tricky to see, but this is eliminating the need for a lot of repetition. For example, before setting this up, this task was delegated to a heap of `switch` statements. Messy! Let's look at how we invoke this and then explain the flow.

<p class="block-header">Example Module Usage</p>

```javascript
Template.content.helpers({
  content: function( contentType ) {
    var content = Modules.both.getContent({
      type: contentType,
      method: 'find',
      query: {},
      projection: { fields: { "title": 1, "slug": 1 }, sort: { "title": 1 } }
    });

    if ( content ) {
      return content;
    }
  }
});
```
We invoke the function `Modules.both.getContent` passing an object of options. Here we've passed the type of content we want `contentType` (returned from another helper), the Mongo method we want to call, the query we want to pass, and then the projection. This may seem like a bit much but it solves a much bigger problem: querying for multiple content types at once. Notice that the helper we're returning to is _not_ specific to the content type. Instead, we delegate that task to our module, expecting it to return the right content based on the context and parameters we pass it.

Woah! What's neat about this is that it allows us to reduce our code significantly and make our application easier to reason about. That namespace isn't a fluke. It's purposeful. If another developer were to look at this code, it's easy to reason that the example above lives somewhere like `/both/modules/get-content.js`. Not only does this pattern reduce code, it also makes it easier to find what code is responsible for what functonality!

<div class="note success">
  <h3>Uh, okay. But why? <i class="fa fa-thumbs-up"></i></h3>
  <p>Beyond organization and reusability, this pattern is also preferred because it maps well to the <a href="https://babeljs.io/docs/learn-es2015/#modules">Modules feature of ES2015</a>. Metoer doesn't support this right now, but with plans to do so in the future, it makes sense to get our code as close as possible right now. When support is enabled, we can simply add an export and import statement to our files where necessary. Our modules will function exactly the same!</p>
</div>

### /client
Next up we have our `/client` directory where we store all of our client-side code. This is pretty basic. We have six sub-directories: `helpers`, `lib`, `modules`, `routes`, `stylesheets`, and `templates`. `/client/helpers` is designed to hold helper code. In most cases, this contains [global template helper](https://docs.meteor.com/#/full/template_registerhelper) definitions. Sometimes, though, additional helper files are added for things like manipulating text, so it's nice to give helpers their own directory.

`/client/lib` is added to hold JavaScript code that needs to load on the client first. Notice, inside `lib` is another directory `vendor`. This is where I store things like JavaScript libraries that haven't been added as Meteor packages. This is helpful when a library you need isn't available as a package and you don't want to wrap it yourself.

`/client/modules` serves the exact same purpose as we outlined above, however, modules defined here are intended for use only on the client. `/client/routes` contains routing definitions in two groups: `authenticated` and `public`. This is a personal convention that helps me understand how and when routes are loaded.

This pattern is specific to Iron Router at this point, _not_ Flow Router. Due to conventions introduced by Flow Router and its usage of Fast Render, it requires that you place your routes in the `/both` (they label it as `/lib`) directory. As support moves toward Flow Router, this `routes` directory will likely be moved to `/both` to account for this.

`/client/stylesheets` simply contains stylesheets. I prefer to use [Sass](http://sass-lang.com/) and so this directory usually contains a single `application.scss` file which is used to import stylesheets from sub-directories within `/client/stylesheets`. This could easily contain plain CSS files, too. Finally, we have a `/client/templates` folder which contains two sub-directories: `authenticated` and `public`. The logic here is to break up our templates based on the context where they're used. Inside each, we store a single `template-name.html` file and a `template-name.js` file to contain the template markup and logic, respectively.

Also in `/client` is a single file `startup.js`. This is designed to hold any code that needs to run when the client starts up like configuration. It's optional and isn't always used, but it's good to have by default so your startup code has a place to live.

### /collections
This one is straightforward. Here, we store a single JavaScript file for _each_ collection we define. For example, given a collection called "Documents", we'd add a file to this directory called `documents.js`. The reason this is done is that all of the code pertaining to a collection definition—the global variable definition, allow/deny rules, schema, etc—are stored in the same file. This makes it easy to come back later and see how a collection is taking shape and being manipulated.

### /private
This directory is used sparingly. Inside you'll see a single directory `email-templates` as this is usually [where email templates are stored](https://themeteorchef.com/snippets/using-the-email-package/#tmc-sending-html-email-using-templates). Specifically, these are HTML templates that are used to send HTML email on the server. On occasion, `/private` will also contain raw data files like `seed.json` to store data that will be used to seed the database for testing purposes.

### /public
Used for its intended purpose: storing public files. Generally I try to make sure that any static assets are stored on a CDN or something like [Amazon S3](https://aws.amazon.com/s3), but on smaller projects this will hold things like images, font files, and miscellaneous graphics. In most cases, this will have a single file in it `favicon.ico` containing the favicon for the application.

### /server
The server is kept fairly lean. Just two directories: a `/server/modules` directory for storing any module code and a `/server/publications` directory for storing publication definitions. For publications, with the introduction of [template-level subscriptions](https://docs.meteor.com/#/full/Blaze-TemplateInstance-subscribe), I've started creating single files based on the template name. This helps us to see where a publication is intended to be subscribed to. So, for example, if we had a template `dashboard.html`, in it's `onCreated` event we'd call `this.subscribe( 'dashboard' );` subscribing to a publication defined at `/server/publications/dashboard.js`. Nifty!

Similar to the `/client` directory, we also include a `startup.js` file for code that needs to run when the server starts up. Note: code in this file is intended to be wrapped in a `Meteor.startup( function() { // startup code here });` block.

### /tests
Last but not least, our dear ol' friend `/tests`. This behaves exactly as it was described above: storing tests. Following the conventions of the [Velocity framework](https://velocity.readme.io/), this usually contains a folder like `/tests/jasmine` which then contains several sub-folders for the test types (e.g. `/tests/jasmine/client/unit`). I've only recently started testing my code on a project-level basis so I'm still learning how to organize this well.

### Root files
In the root of the project we'll find four files: `application.html`, `package.json`, `settings-development.json`, and `settings-production.json`. `application.html` contains the `<head>` and `<body>` code for our application as a whole like this:

<p class="block-header">application.html</p>

```markup
<head>
  <title>Application Title</title>
  <!-- Any head-related matter goes here -->
</head>

<body></body>
```
`package.json` follows [the NPM convention](https://docs.npmjs.com/files/package.json) and stores some information about the project and more importantly, a set of NPM scripts used for working on the project. This is generally used in conjunction with the other two files here: `settings-development.json` and `settings-production.json`. These two files represent our settings files for the `development` and `production` environments respectively. `settings-development.json` stores values that we use during development, while `settings-production.json` stores values that we use _in production_.

The reason we have two is that `-development.json` is committed to source so other developers can access it, while `-production.json` is _not_ committed to source (for [security reasons](https://themeteorchef.com/snippets/making-use-of-settings-json/#tmc-settingsjson-in-development-vs-production)). Having two files, we can [use NPM scripts](https://themeteorchef.com/snippets/making-use-of-settings-json/#tmc-automating-settingsjson) to control how and when the files are loaded.

That wraps up the basic starting point used at The Meteor Chef. Again, this is just coming together so it's not in use in things like recipes or [Base](https://github.com/themeteorchef/base) just yet. Expect to see more of this pattern soon!

### Meteor example
In the Meteor documentation, we can find [an example](https://docs.meteor.com/#/full/structuringyourapp) of how to organize our project. Keep in mind, this isn't an official organization pattern, merely an example that we can work with.

```coffeescript
lib/                        # common code like collections and utilities
lib/methods.js              # Meteor.methods definitions
lib/constants.js            # constants used in the rest of the code

client/compatibility        # legacy libraries that expect to be global
client/lib/                 # code for the client to be loaded first
client/lib/helpers.js       # useful helpers for your client code
client/body.html            # content that goes in the <body> of your HTML
client/head.html            # content for <head> of your HTML: <meta> tags, etc
client/style.css            # some CSS code
client/<feature>.html       # HTML templates related to a certain feature
client/<feature>.js         # JavaScript code related to a certain feature

server/lib/permissions.js   # sensitive permissions code used by your server
server/publications.js      # Meteor.publish definitions

public/favicon.ico          # app icon

settings.json               # configuration data to be passed to meteor --settings
mobile-config.js            # define icons and metadata for Android/iOS
```

One file to point out here that's _not_ included in the TMC example above is `mobile-config.js`. This file—with this exact name—is what you would use to [configure your application](http://docs.meteor.com/#/full/mobileconfigjs) if it needed to run on iOS or Android. Beyond this, this pattern is pretty simplistic. It uses some different naming conventions for files but doesn't make too many suggestions about how to organize your code.

### Unofficial Meteor FAQ
One of the first community-offered solutions for organizing Meteor applications, the Unofficial Meteor FAQ's answer to ["Where I should put my files?"](https://github.com/oortcloud/unofficial-meteor-faq#where-should-i-put-my-files) was the first guide I used to organize my applications.

```coffeescript
lib/                       # <- any common code for client/server.
lib/environment.js         # <- general configuration
lib/methods.js             # <- Meteor.method definitions
lib/external               # <- common code from someone else

## Note that js files in lib folders are loaded before other js files.

collections/               # <- definitions of collections and methods on them (could be models/)

client/lib                 # <- client specific libraries (also loaded first)
client/lib/environment.js  # <- configuration of any client side packages
client/lib/helpers         # <- any helpers (handlebars or otherwise) that are used often in view files

client/application.js      # <- subscriptions, basic Meteor.startup code.
client/index.html          # <- toplevel html
client/index.js            # <- and its JS
client/views/<page>.html   # <- the templates specific to a single page
client/views/<page>.js     # <- and the JS to hook it up
client/views/<type>/       # <- if you find you have a lot of views of the same object type
client/stylesheets/        # <- css / styl / less files

server/publications.js     # <- Meteor.publish definitions
server/lib/environment.js  # <- configuration of server side packages

public/                    # <- static files, such as images, that are served directly.

tests/                     # <- unit test files (won't be loaded on client or server)
```
Again, pretty simple. Being the first source of inspiration for my work on TMC, you can see some similar conventions here. Something unique about this is the use of `/lib` in the root, as well as in the `/client` and `/server` directories. Beyond this, there's also an addendum to this list in respect to developing individual features:

```coffeescript
feature-foo/               # <- all functionality related to feature foo
feature-foo/lib/           # <- common code
feature-foo/models/        # <- model definitions
feature-foo/client/        # <- files only sent to the client
feature-foo/server/        # <- files only available on the server
```

This is suggested as a temporary solution. As it's understood, the thinking here is to store each of these directories in the root of your project and later, refactor them into individual Meteor packages. It's explained that using this pattern, each feature's internals follow the same pattern as the pattern for the root above.

### Other examples
This list could go on for awhile. The reality is that there are _a lot_ of different ways to organize projects. Below is a list of additional examples that you can pull ideas from and adapt for your own projects!

- [Meteor Boilerplate](https://github.com/matteodem/meteor-boilerplate) by Matteo De Micheli
- [Void](https://github.com/SachaG/Void) by Sacha Grief
- [Differential Boilerplate](https://github.com/Differential/meteor-boilerplate) by Differential
- [Opinionated Skeleton](https://github.com/jamesdwilson/meteor-jw-opinionated-skeleton) by James Wilson
- [Iron CLI](https://github.com/iron-meteor/iron-cli) by Chris Mather

This is definitely just skimming the surface. If you know of a boilerplate or organization convention you enjoy (or have your own), please share in the comments below!

## Takeaways
- Meteor has a handful of conventions in the form of Special Directories. Make sure to understand these and incorporate them into your project as needed.
- There are a lot of ways to organize your application. Be creative! Mix and match what others are doing to meet your own needs.
- Experiment! You will learn new things as you continue to work with Meteor. Don't be afraid to evolve your project structure as you develop your own preferences.
