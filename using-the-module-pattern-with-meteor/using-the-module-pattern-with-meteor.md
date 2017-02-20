In this snippet, we'll learn how to use the module pattern to organize our code. We'll learn about how to configure our app to use the module pattern, look at some examples of working with modules, and discuss the benefits of using modules to improve the organization and clarity of our code.

When it comes to developing complex applications, we often have to manage several complex processes. Processes where we have to perform several related tasks, all at once. For example, if we're creating a signup flow for our SaaS app, the actual process may involve storing a credit card, creating a user in the database, adding the user to our mailing list, and starting a subscription on a payment service. While each of these are what we might call "discrete steps," they're all related. How do we keep them organized?

Enter [the module pattern](https://toddmotto.com/mastering-the-module-pattern/), a design pattern that helps us solve this multi-step problem by organizing our code into "modules" made up of individual, related functions. In this snippet, we're going to take a look at how to set up our application to make use of modules and then showcase some examples of writing modules.

## Defining namespaces
In order to get this pattern working in Meteor, we need to define a "namespace" for our modules to be assigned to. The basic idea behind a namespace is that it exists as a global object that we assign methods to. Because it's global, we can assign methods to that object from separate files. Here's an example of this happening from within a single file using an object literal:

```javascript
Modules = {
  both: {
    add( n1, n2 ) {
      return n1 + n2;
    }
  },
  client: {
  	subtract( n1, n2 ) {
      return n1 - n2;
    }
  },
  server: {
    multiply( n1, n2 ) {
      return n1 * n2;
    }
  }
};
```

Here, then, we can call `Modules.client.subtract( 10, 5 );` and get back `5` as an answer. This sort of organization is great, but we want to be able to define methods in their own files. To do it, we need to add a bit of structure to our app in the form of directories and config files.

<p class="block-header">/both/modules/_modules.js</p>

```javascript
Modules      = {};
Modules.both = {};
```

<p class="block-header">/client/modules/_modules.js</p>

```javascript
Modules.client = {};
```

<p class="block-header">/server/modules/_modules.js</p>

```javascript
Modules.server = {};
```

Pay attention to the directory paths here. We're defining our namespaces in different directories on purpose. To be clear, here's how this would look in a file tree:

```javascript
/both
--- /modules
------ _modules.js
/client
--- /modules
------ _modules.js
/server
--- /modules
------ _modules.js
```

Cool! So how does this work? The most important file here is `/both/modules/_modules.js`. Notice that in this file, we're defining an object `Modules = {};`. The trick at play here is that in a Meteor app, any folder in the root that doesn't match one of [Meteor's "Special Directories"](https://themeteorchef.com/snippets/organizing-your-meteor-project/#tmc-special-directories) is made accessible to _both_ the client _and_ the server.

Taking advantage of this, we create a directory called `/both`. From here, we can expect that any code in this file is executed on both the client and the server. Inside, then, we define a directory `/modules` and inside of _that_ a file called `_modules.js`. We're really taking advantage of Meteor's loading patterns here.

Specifically, we're relying on Meteor's preference to load the most deepest files in a directory first, followed by the preference to load files alphabetically. Because we're using the `_` (underscore) character to prefix our `_modules.js` file, it gets loaded first (we'll see why this is important in a little bit). Inside of this file, we're defining a global variable `Modules` equal to an empty object and then immediately nesting another object `Modules.both` _within_ that object.

If we `console.log( Modules )` on the client, we can expect to see something like:

<p class="block-header">Browser Console</p>

```javascript
=> Object { both: Object, client: Object };
```

![Logging Modules to the browser console.](https://cl.ly/image/1g1u1z253N40/Image%202015-09-22%20at%209.00.52%20PM.png)

And then, if we do the same thing on the server...

<p class="block-header">Server Console</p>

```javascript
=> { both: {}, server: {} };
```

![Logging Modules to the server console using Meteor shell.](https://cl.ly/image/0Y1v3Z1t0t1Y/Image%202015-09-22%20at%209.03.27%20PM.png)

Making some sense? So, code stored at `Modules.both` can be seen on both the client and the server, but code stored at `Modules.client` is only visible on the client and code stored at `Modules.server` is only visible on the server! How cool is that? With this in place, then, we can start to define modules.

## Defining modules
Defining modules is pretty easy. The idea is that we want to create a file with a name that matches the name of our module in the folder where that module needs to be accessed. A quick example, imagine we wanted to define a module called `getTacos` that was accessible only on the client:

<p class="block-header">/client/modules/get-tacos.js</p>

```javascript
Modules.client.getTacos = {};
```

Pretty simple, right? And if we wanted this accessible just on the server:

<p class="block-header">/server/modules/get-tacos.js</p>

```javascript
Modules.server.getTacos = {};
```

And it should be clear, but if we wanted this accessible on both the client and the server:

<p class="block-header">/both/modules/get-tacos.js</p>

```javascript
Modules.both.getTacos = {};
```

Neat! If we wanted we could stop here and just define our methods on an object. What we want to avoid, though, is unnecessarilly exposing functions related to our method outside of our file. For example, using plain object literals like the examples above, we could technically access _any_ methods defined on those objects. Taking our client example:

<p class="block-header">/client/modules/get-tacos.js</p>

```javascript
Modules.client.getTacos = {
  publicMethod() {
    console.log( "I should be accessible by anyone." );
  },
  privateMethod() {
    console.log( "I should NOT be accessible by anyone." );
  }
};
```

As-is, even though we don't want our `privateMethod` accessible, we can easily pop open our browser console and get access to it.

<figure>
  <img src="https://cl.ly/image/221P3H1t3P2G/object-literal-exposure.gif" alt="An object literal exposing all of its methods.">
  <figcaption>An object literal exposing all of its methods.</figcaption>
</figure>

Bummer! How do we get around this? Good news: it's really easy.

### Designing modules that have private functions
Something that's interesting about Meteor is how it bundles up our code. All JavaScript in Meteor is wrapped in a closure automatically, like this:

<p class="block-header">/client/modules/get-tacos.js</p>

```javascript
(function(){

/////////////////////////////////////////////////////////////////////////
//                                                                     //
// client/modules/get-tacos.js                                         //
//                                                                     //
/////////////////////////////////////////////////////////////////////////
                                                                       //
Modules.client.getTacos = {                                            // 1
  publicMethod: function () {                                          // 2
    console.log("I should be accessible by anyone.");                  // 3
  },                                                                   //
  privateMethod: function () {                                         // 5
    console.log("I should NOT be accessible by anyone.");              // 6
  }                                                                    //
};                                                                     //
/////////////////////////////////////////////////////////////////////////

}).call(this);
```

This is the actual output of our `/client/modules/get-tacos.js` file after being bundled up by Meteor. Notice that aside from the [source mapping](https://blog.teamtreehouse.com/introduction-source-maps) comments, the `(function(){ /* Our code */ }).call( this );` wrapping our code. This is an [immediately invoked function expression](http://benalman.com/news/2010/11/immediately-invoked-function-expression/) (IIFE), meaning, as soon as this file loads, the code inside it runs. If instead we updated this file to simply include a `console.log( "Modules.client.getTacos" );`, as soon as our app loaded, we'd see `"Modules.client.getTacos"` printed to the console. Wild!

So...what exactly does this get us? Well, because Meteor is wrapping our file in an IIFE (pronounced "iffy"), any variables that are defined _locally_ within our file are automatically private, while global variables remain global. Hmm. Let's refactor our `getTacos` module a bit so that our `privateMethod` is actually private.

<p class="block-header">/client/modules/get-tacos.js</p>

```javascript
let publicMethod = () => {
  console.log( "I should be accessible by anyone" );
};

let privateMethod = () => {
  console.log( "I should NOT be accessible by anyone" );
};

Modules.client.getTacos = publicMethod;
```

Now, if we call `Modules.client.getTacos()` from the browser console...

<figure>
  <img src="https://cl.ly/image/2C0N3Z2m2W08/private-methods-in-module.gif" alt="Our private method is now invisible!">
  <figcaption>Our private method is now invisible!</figcaption>
</figure>

Neat. So now, we can't access the `privateMethod` method from anywhere but the file it's been defined in. Notice that this happens because we never "expose" our `privateMethod`. Instead, we define one function as our "public" method which can get access to our private methods without the need to expose them globally. This is where the module pattern gets really interesting.

### Breaking complex tasks into multiple functions
The real power of modules is our ability to define several, smaller functions that accomplish individual tasks, pulled together into one, easy to call method. For example, let's take an example of a module that takes in a series of arguments and uses private methods to help us return a value.

<p class="block-header">/client/modules/get-tacos.js</p>

```javascript
let _getCheeseCalories = ( cheese ) => {
  var cheeses = {
    "cheddar": {
      calories: 120
    },
    "queso": {
      calories: 200
    }
  };

  return cheeses[ cheese ].calories;
};

let _getMeatCalories = ( meat ) => {
  var meats = {
    "steak": {
      calories: 50
    },
    "chicken": {
      calories: 100
    },
    "beef": {
      calories: 200
    }
  };

  return meats[ meat ].calories;
};

let _getGuacamoleCalories = ( guacamole ) => {
  return 45;
};

let _addCalories = ( cheeseCal, meatCal, guacamoleCal ) => {
  return cheeseCal + meatCal + guacamoleCal;
};

let calories = ( options ) => {
  let cheeseCalories    = _getCheeseCalories( options.cheese ),
  	  meatCalories      = _getMeatCalories( options.meat ),
      guacamoleCalories = options.guacamole ? _getGuacamoleCalories() : 0;

  return _addCalories( cheeseCalories, meatCalories, guacamoleCalories );
};

Modules.client.getTacoCalories = calories;
```

In this example, our goal is to pass an object of ingredients to a method called `getTacoCalories`. We want to get back a numeric value representing the total number of calories for a taco containing those ingredients. Albeit a bit ridiculous, this example demonstrates how we can break individual tasks—getting calories for different types of ingredients—into smaller, easier to understand functions.

The power of this is that it allows us to turn complex processes into easily repeated processes. Here, we may have to get calories for different combinations of tacos. Following this pattern we can call `Modules.client.getTacoCalories({ meat: "steak", cheese: "queso", guacamole: true });` and get a value back, or, `Modules.client.getTacoCalories({ meat: "beef", cheese: "cheddar", guacamole: false });` and get a _different_ value back. To show this off a bit:

<figure>
  <img src="https://cl.ly/image/1b472p300f0G/getting-taco-calories.gif" alt="Reusing the same module to get different values.">
  <figcaption>Reusing the same module to get different values.</figcaption>
</figure>

It's important to point out, too, that the only method that's publicly accessible is `calories`, here, because it's assigned to our global namespace. Everything else is locked up in our file thanks to the IIFE Meteor wraps our code with combined with locally scoped variables!

### A more Meteor-y example
So, getting taco calories is cool and all, but what if we're working with _real_ Meteor code? Yeah, yeah. Okay. Here's a more complex example showcasing something close to the example we suggested at the start of this snippet (creating a new user, a payment profile for them, and subscribing them to a mailing list).

<p class="block-header">Realistic Module Example</p>

```javascript
var Stripe            = StripeAPI( Meteor.settings.private.stripe.secret ),
    MailChimpSettings = Meteor.settings.private.mailchimp,
    mailchimp         = new MailChimp( MailChimpSettings.apiKey, { version: '2.0' } );

var _createCustomerOnStripe = function( email, plan, source ) {
  var createCustomer = Meteor.wrapAsync( Stripe.customers.create, Stripe.customers ),
      getCustomerId  = createCustomer({
        email: email,
        plan: plan,
        source: source
      });

  return getCustomerId.id;
};

var _createUser = function( email, password ) {
  return Accounts.createUser( { email: email, password: password } );
};

var _createCustomerInDatabase = function( userId, customerId ) {
  Customers.insert({
    "user": userId,
    "stripeCustomerId": customerId,
    "subscription": {
      "active": true,
      "cancelling": false
    }
  });
};

var _subscribeToMailingList = function( email ) {
  mailchimp.call( 'lists', 'subscribe', {
    id: MailChimpSettings.listId,
    email: {
      email: email
    }
  });
};

var create = function( options ) {
  var customer = _createCustomerOnStripe( options.email, options.plan, options.stripeToken ),
      user     = _createUser( options.email, options.password );

  _createCustomerInDatabase( user, customer );

  if ( options.canEmail ) {
    _subscribeToMailingList( options.email );
  }
};

Modules.server.createCustomer = create;
```

Prett neat, right? We won't step through it here, but you're encouraged to tweak and play with this example! The advantage should be clear: instead of digging through a long river of nested callbacks, we can cleary see each step, the order it's called in, and the intended result _from_ that work. [Shred](https://youtu.be/3bklFIanHdk?t=1m47s).

## Takeaways
- Modules can help you to organize complex processes into clear, repeatable functions that can be called again and again.
- Using modules can help you "hide" functions that you don't want exposed publicly.
- Modules make it much easier for other developers to read your code because they're broken into small, easy-to-follow chunks.
