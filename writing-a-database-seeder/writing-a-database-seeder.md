In this recipe, we'll learn how to automate the process of getting test data into our app by writing a database seeder. We'll build an ES2015 class that helps to handle the process of creating users, as well as inserting into any collection we choose. We'll learn how to seed a collection using our own data, as well as randomly generating data using the library Faker. 

<div class="note info">
  <h3>Pre-Written Code <i class="fa fa-info"></i></h3>
  <p><strong>Heads up</strong>: this recipe relies on some code that has been pre-written for you, <a href="https://github.com/themeteorchef/writing-a-database-seeder">available in the recipe's repository on GitHub</a>. During this recipe, our focus will only be on implementing a database seeder on the server. If you find yourself asking "we didn't cover that, did we?", make sure to check the source on GitHub.</p>
</div>

<div class="note">
  <h3>Additional Packages <i class="fa fa-warning"></i></h3>
  <p>This recipe relies on several other packages that come as part of <a href="https://github.com/themeteorchef/base">Base</a>, the boilerplate kit used here on The Meteor Chef. The packages listed below are merely recipe-specific additions to the packages that are included by default in the kit. Make sure to reference the <a href="https://themeteorchef.com/base/packages-included/">Packages Included list</a> for Base to ensure you have fulfilled all of the dependencies.</p>
</div>

## Prep
- **Time**: ~1-2 hours
- **Difficulty**: Intermediate
- **Additional knowledge required**: [ES2015](https://themeteorchef.com/blog/what-is-es2015/) [basics](https://themeteorchef.com/snippets/common-meteor-patterns-in-es2015/)

## What are we building?
BrainBots, LLC is a tiny web consultancy focused on developing Meteor JavaScript applications for clients. They're a team of two working out of a co-working space in downtown Orlando, Florida. Because the founders of BrainBots—Henrik and Sofia—are also the people doing the work, they have to be as efficient as possible when working on client projects.

Recently, they noticed that testing out how data is displayed in the app is difficult. Every time they need to reset their database, they have to spend some non-trivial amount of time typing in data via forms. It's become so time consuming that they're having trouble juggling multiple clients and it's affecting their cash flow.

The BrainBots team has asked us to help them build a database seeder to help automate the process of getting test data into their client apps. We'll help them build a way to generate test data based on information that they provide as well as randomized data that can be created based on a simple script. They've asked that we set this up to be as reusable as possible so that they only have to specify their test data at the start of a project (or when they add new resources).

To get the job done, we'll be focused on writing a server-side class that can be reused with multiple collections. When all is said and done, Henrik and Sofia will be able to use the following functions to handle seeding of their project databases:

<p class="block-header">With specific data</p>

```javascript
Seed( 'Users', {
  data: [
    {
      email: 'henrik@brainbots.fm',
      password: 'password',
      profile: {
        name: { first: 'Henrik', last: 'Müller' }
      },
      roles: [ 'admin' ]
    },
    {
      email: 'sofia@brainbots.fm',
      password: 'password',
      profile: {
        name: { first: 'Sofia', last: 'Carmichael' }
      },
      roles: [ 'admin' ]
    }
  ]
});
```

<p class="block-header">Without specific data</p>

```javascript
Seed( 'Products', {
  min: 5,
  environments: [ 'development', 'staging', 'product' ],
  model( index ) {
    return {
      name: faker.commerce.product(),
      price: faker.commerce.price()
    };
  }
});
```

## Ingredients
Before we start building, make sure that you've installed the following packages and libraries in your application. We'll use these at different points in the recipe, so it's best to install these now so we have access to them later.

### Meteor packages

<p class="block-header">Terminal</p>

```bash
meteor add digilord:faker
```

We'll rely on the `digilord:faker` package to give us access to the [Faker](https://github.com/Marak/faker.js) library for generating fake data.

<p class="block-header">Terminal</p>

```bash
meteor add alanning:roles
```
We'll use the `alanning:roles` package to give us (optional) support for applying roles to any users that we seed the database with.

## Defining our seeder class
For this recipe, we're going to rely on an [ES2015 Class](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes). By using a class, we'll gain a little bit of structure for managing the seeding process. To get started, let's wire up a file in the root of our `/server` directory called `seed.js`. In here, we'll define our class as well as make it globally accessible for Henrik and Sofia on the server. Let's start by giving our class a skeleton and work up from there!

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor( collection, options ) {
    // Work with our arguments here.
  }
}

Seed = ( collection, options ) => {
  return new Seeder( collection, options );
};
```

To get started we're keeping things pretty simple. First, we define a class `Seeder` and set it up with a constructor function expecting two arguments: `collection` and `options`. Per our conversation with Henrik and Sofia, they asked if we could make this work for _any_ collection simply by passing the collection name. Here, by accepting `collection` as an argument, we'll make this possible. For `options`, we'll expect an object with a handful of properties that will let the seeder know what it should do.

Down below, we've got something unique going on. To clean up the user experience of working with our seeder, we've decided to wrap its invoication in a function set to a global variable called `Seed`. The idea here is that instead of having to call `new Seeder( 'Collection', { /* options */ } )` every time we use the class, we just call `Seed( 'Collection', /* options */ } )`. A few keystrokes saved doesn't seem like much, but this gives us a nice clean API for interacting with our class. With that in mind, notice that we simply pass along the `collection` and `options` arguments we'll get back from calling `Seed()` to a new instance of our `Seeder` class.

### Wiring up our `constructor()`
With this basic outline in place, the next thing we need to do is wire up our constructor. That name is a bit ambigious...what does it do? `constructor()` is a special method that's built into classes and is responsible for "creating and initializing an object created with a class." Said in non-dorky terms, this is where we "set up" our class. Here, we can define things like variables that are accessible in all of the other methods defined on our object as well as _call_ other methods on our class. In just a bit, we'll use the `constructor()` method to kick off the actual seeding process.

For now, let's add a little more structure to our constructor. Our goal is to prevent any unwanted behavior from taking place. Our first stop is ensuring that we have the appropriate arguments in place!

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor( collection, options ) {
    if ( !collection || !options ) {
      throw new Error( 'Please supply a collection to seed and options for seeding. Usage: Seed( collectionName, options ).' );
    } else {
      // Kick off the seeding process.
    }
  }
}

[...]
```

Here, we're setting up our class to ensure that it has both a collection name passed as well as a set of options. If one (or both) are missing, our constructor will stop our call and throw an error. This is important because we want to ensure that Henrik and Sofia are alerted when they use the `Seed()` function improperly.

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor( collection, options ) {
    if ( !collection || !options ) {
      throw new Error( 'Please supply a collection to seed and options for seeding. Usage: Sow( collectionName, options ).' );
    } else {
      this.collection = this.getCollection( collection );
      this.options    = options;
    }
  }
}

[...]
```

Ah, yes! There's the scary stuff. Once we've confirmed that we have our arguments, our next task is to start to figure out what exactly we're doing. The first step in that process is one of the most important: getting access to the collection we're trying to seed. For this, we're going to add another method to our class `getCollection()`. Notice that to invoke it, we simply call `this.getCollection()`. How does that work?

This is the neat part about classes: all of the methods defined on them are accessible via `this` in _all other methods_. The same goes for properties, too. Notice that just below our call to `this.collection = this.getCollection( collection )` we're setting `this.options` equal to the `options` argument we passed in when we originally invoked our class. Now, `this.options` will be equal to whatever object gets passed in when we call `Seed( 'Collection', { /* I become this.options */ } );`

Before we get too far ahead of ourselves, let's take a peek at `this.getCollection()` to see what it's up to. Pay close attention, it's expecting the `collection` argument we received during invocation.

### Getting the collection
In Meteor, working with collections can be a little bit tricky. If we know the global variable of the collection we're trying to access things are pretty straightforward. But how about when our collection name is dynamic? We have a few ways of solving this problem. The first is to simply pass the global variable for our collection e.g., `Documents`. Another—and slightly more fool proof version—is to use Meteor's `global` object on the server.

The `global` object in Meteor—available only on the server—contains _tons_ of information about our project. It's responsible for storing all of the global values in our application. The good news is that it also stores the global variables for each of our collections. Huh? This means that we can access our collections not just using a global variable like `Documents` on the server, but also by calling `globals[ 'Documents' ]`. Pretty neat, eh?

In our case, we want Henrik and Sofia to be able to pass a string value that's either lowercase or capitalized and get back the appropriate collection instance. To do it, we have two steps: ensure that we're capitalizing the string value we're passed in the call to `Seed( 'collectionName' )` as the `global` object contains the actual global variable defined for the collection. To make sure that's clear, assume the following collection definition:

<p class="block-header">/collections/pizza.js</p>

```javascript
Pizza = new Mongo.Collection( 'pizza' );
```

Here, the variable `Pizza` is accessible on the server via `global[ 'Pizza' ]` or `global.Pizza`. Neat, huh?! In our case, we want to be able to pass either `pizza` or `Pizza` and get the same result. Let's add an additional method to our class now `getCollection` and see how this is all wired up.

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor( collection, options ) {
    if ( !collection || !options ) {
      [...]
    } else {
      this.collection = this.getCollection( collection );
      this.options    = options;
    }
  }

  getCollection( collection ) {
    let collectionName = this.sanitizeCollectionName( collection );
    return collectionName === 'Users' ? Meteor.users : global[ collectionName ];
  }

  sanitizeCollectionName( collection ) {
    return collection[ 0 ].toUpperCase() + collection.slice( 1 );
  }
}
```

Okay, I lied! We're actually adding _two_ methods: `getCollection` and `sanitizeCollectionName`. The first does what it say: gets us back the appropriate collection. Before we get the collection, however, we need to account for the possibility that a collection name has been passed to us as a lowercase value. Inside of `sanitizeCollectionName()`, we're taking the passed collection name and returning the capitalized version of it. So, if our `collection` argument is equal to `pizza`, we'd get back `Pizza` in return. Neat!

Once we have this, back up in our `getCollection` method we've got a dash of logic to contend with. Here, we check to see whether or not our capitalized collection name is equal to `Users`. Why's that? Well, sneaky folks they are, the Meteor team defines the `users` collection as `Meteor.users`. Technically, `Meteor.users` is just an alias for `Accounts.users`! Because of this, if we were to call `global[ 'Users' ]` we'd get nothing back. By testing here, we make a preemptive check to return `Meteor.users` if the passed value is in fact `Users`.

If it's not, though, we bust out our `global` object, passing in the capitalized version of our collection name using bracket syntax (this allows us to keep this as a variable). Sweet! Now we have access to the passed collection as long as we get a string representing either the lowercase or capitalized version of the collection's global variable. So far so good. Let's hop back up to our `constructor()` method to kick off the rest of our seeding process.

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor( collection, options ) {
    if ( !collection || !options ) {
      throw new Error( 'Please supply a collection to seed and options for seeding. Usage: Sow( collectionName, options ).' );
    } else {
      this.collection = this.getCollection( collection );
      this.options    = options;

      if ( typeof this.collection !== 'undefined' ) {
        this.seed();
      } else {
        throw new Error( `Sorry, couldn't find the collection "${ collection }" to seed!` );
      }
    }
  }

  [...]
}
```

See what we're up to here? Once we have our collection back from `this.getCollection()`, we assign it to `this.collection` so our collection is accessible throughout the entire class. Once that's set, down below we make one last check to ensure that our collection did _not_ come back as `undefined`. If it is `undefined`, we throw an error letting Henrik and Sofia know. If all is well, we kickoff the seeding process by calling `this.seed()`. Can you guess what that is? Yep! Another method. Let's dig into it now.

## Determining a seeder method
Before we actually start the seeding process, we need to decide _which_ type of seeding process we're performing. Remember, earlier we showed an example of two methods for seeding our database: one with a fixed array of data, the other with a simple model that we'd like to clone a certain number of times. This is the point where we make a decision on which path to take based on the options passed to us.

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor() {
    [...]
  }

  seed() {
    let options = this.options,
        data    = options.data,
        model   = options.model;

    if ( data && !model ) { this.sow( data );  }
    if ( model && !data ) { this.sow( model ); }

    if ( options.data && options.model ) {
      throw new Error( `Please choose to seed from either a data collection or a model. Cannot do both!` );
    }
  }
}
```

Pretty simple! We cache a few of our values at the top for a little clarity/performance bump and then dig into some logic. If our `options` object has a `data` property but not a `model` property, we call `this.sow()` passing in `data` as an argument (we'll cover `this.sow()` next). If we have a `model` property but not a `data` property, we call `this.sow()` passing in `model` as an argument. Last but not least, if we find that `Seed()` has been passed both a `data` _and_ `model` property in its `options`, we throw an error letting Henrik and Sofia know that they have to pick one or the other! Why?

As we'll see in a bit, we'll need to control how we loop over the data we're inserting based on whether or not we're using the data (Array) method, or the model (Function returning an Object) method. Technically speaking, either will work perfectly fine without this check (our code will be pretty bulletproof), however, it's good to bark if we try to do both at once. Better safe than sorry!

Next up, let's get into the powerful part of this recipe: `this.sow()`. This method will be responsible for the actual looping of our data and controlling how we get the passed data into the database.

## Looping over our passed data
Pay close attention here! This is where the rubber hits the road. In our `sow()` method, we've got a lot of work to do around determining what type of insertion we're doing, how we're going to do it, and whether or not we're allowed to do it. The method itself is pretty small, but it's got a lot going on. Let's dump out the whole thing and work through the pieces.

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor() {
    [...]
  }

  [...]

  sow( data ) {
    let isDataArray        = data instanceof Array,
        loopLength         = isDataArray ? data.length : this.options.min,
        hasData            = this.checkForExistingData(),
        collectionName     = this.collection._name,
        isUsers            = collectionName === 'users',
        environemntAllowed = this.enviornmentAllowed();

    if ( !hasData && environemntAllowed ) {
      for ( let i = 0; i < loopLength; i++ ) {
        let value = isDataArray ? data[ i ] : data( i );

        if ( isUsers ) {
          this.createUser( value );
        } else {
          this.collection.insert( value );
        }
      }
    }
  }
}
```

It's not as bad as it seems! I promise. Let's walk through it. First up, we need to figure out whether or not the value being passed in for the `data` argument is an `Array` or not. Remember, up above in our `seed()` method, we're potentially passing in either an array or a _function_. This first check let's us know whether or not we're getting an array back.

<div class="note">
  <h3>Oh, JavaScript! <i class="fa fa-warning"></i></h3>
  <p>You may be wondering, "why are we using <code>instanceof</code> instead of <code>typeof</code>?" Well, if you pop open your console and type something like <code>typeof [ 1, 2, 3 ]</code>, guess what you'll get back. Hint: it's not <code>Array</code>. Instead, we get back <code>Object</code>! The reason for this is that technically speaking, array's in JavaScript are built on top of the <code>Object</code> prototype.</p>
</div>

Once we know if we're dealing with an array or not, we use that answer to determine the length of our loop down below. If we get an array back, we only want our loop to run as long as the length of that array. If we get back a function, though, we defer to another option that Henrik and Sofia can pass `this.options.min`. Here, `min` will be a minimum number of "copies" to create of the `model()` that we pass (we'll see how this works in examples down below). Next up is a big one: checking if data already exists. Let's set that method up now as it's one of the more important method's in our entire class.

### Check if data already exists
Our method for checking if data exists is pretty simple but can help us to prevent a lot of hyjinx later. Inside of `checkForExistingData()`, our goal is to determine whether or not the _current_ count of documents in the passed collection is greater than or equal to one of two values: `0` or `this.options.min`. Let's take a look and then talk through it:

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor() {
    [...]
  }

  [...]

  sow( data ) {
    [...]
  }

  checkForExistingData() {
    let existingCount = this.collection.find().count();
    return this.options.min ? existingCount >= this.options.min : existingCount > 0;
  }
}
```

Here, we start our method by making a call to `this.collection.find().count()`. Remember, because we set `this.collection` equal to the result of `this.getCollection()` in our method, we now have access to it throughout the entire class as `this.collection`. Here, we do a `.find()` method to find _all_ of the documents in that collection and then call a `.count()` to get a numerical count back of how many there are. Next, we return the result of seeing whether or not that count is greater than or equal to either the value of `this.options.min`, or (if it's not defined), zero.

What this ultimately tells us is whether or not we've already run this seeder before. Because Henrik and Sofia want to use this method _after_ they've reset a database, we want to first confirm that the database—and more specifically, the passed collection—is empty before inserting anything. Making sense? Let's jump back to our `sow()` method and see what else we're up to.

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor() {
    [...]
  }

  [...]

  sow( data ) {
    let isDataArray        = data instanceof Array,
        loopLength         = isDataArray ? data.length : this.options.min,
        hasData            = this.checkForExistingData(),
        collectionName     = this.collection._name,
        isUsers            = collectionName === 'users',
        environemntAllowed = this.enviornmentAllowed();

    if ( !hasData && environemntAllowed ) {
      for ( let i = 0; i < loopLength; i++ ) {
        let value = isDataArray ? data[ i ] : data( i );

        if ( isUsers ) {
          this.createUser( value );
        } else {
          this.collection.insert( value );
        }
      }
    }
  }

  [...]
}
```

Moving right along! Next, we grab the name of the collection (this is the actual name from the MongoDB instance, _not_ the name passed in by our users) and then we use it to determine whether or not this is the `users` collection. Just before we get into our loop we have one more method being defined: `this.environmentAllowed()`. Can you take a guess at what this is doing? Let's take a look!

### Controlling seeding per environment
One of the more important features of our seeder—a surprise feature for Henrik and Sofia—is the ability to control seeding based on _environment_. By environment, we mean whether or not our app is running on our laptop, or if it's deployed to a staging or production environment. In our `this.environmentAllowed()` check, we verify whether or not seeding is allowed on the machine where this code is currently being run.

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor() {
    [...]
  }

  [...]

  sow( data ) {
    [...]
  }

  [...]

  environmentAllowed() {
    let environments = this.options.environments;

    if ( environments ) {
      return environments.indexOf( process.env.NODE_ENV ) > -1;
    } else {
      return true;
    }
  }  
}
```

Simple enough! Here, we begin by pulling the value of `this.options.environments` to determine whether or not we're even concered with environments. If we _are_, we expect `this.options.environments` to return an array of strings like `[ 'development', 'staging', production ]`. If we get back an array, our test is to see whether or not the current value of `process.env.NODE_ENV` is in that array. Wait...what is that? `process.env.NODE_ENV` is an _environment variable_ that returns a string of the application's current location.

By default in Meteor apps running on `localhost`, this is set to `development`. In our staging and production environments, we can set this by [configuring environment variables](https://themeteorchef.com/recipes/deploying-to-modulus/#tmc-environment-variables). The important part is that this method will return `true` only if the current environment is in the passed array. If not, it will return `false` preventing any seeding from taking place. As a fail-safe, if an environment array _hasn't_ been passed, we assume that all environments should be seeded and return `true` to say "yes, this environment is allowed."

Getting excited? Let's jump back up to our `sow()` method and see how we're _finally_ handling inserts.

## Handling inserts
Time for the big show! Up until this point, we've successfully built out nearly _all_ of the methods for our seeder. There's just one left! Before we cover it, though, let's head back up to our `sow()` method and explore how it's managing the looping process.

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor() {
    [...]
  }

  [...]

  sow( data ) {
    let isDataArray        = data instanceof Array,
        loopLength         = isDataArray ? data.length : this.options.min,
        hasData            = this.checkForExistingData(),
        collectionName     = this.collection._name,
        isUsers            = collectionName === 'users',
        environmentAllowed = this.environmentAllowed();

    if ( !hasData && environmentAllowed ) {
      for ( let i = 0; i < loopLength; i++ ) {
        let value = isDataArray ? data[ i ] : data( i );

        if ( isUsers ) {
          this.createUser( value );
        } else {
          this.collection.insert( value );
        }
      }
    }
  }

  [...]
}
```
Phew! With all of that config out of the way, let's do some loopin'. Making use of our earlier work, we make a quick check before starting our loop to ensure that the collection we're seeding _doesn't_ have any data and that the current environment is allowing seeding of the database. If all lights are green, we set up a simple `for` loop, setting our iterator count equal to `loopLength`. Remember that earlier, we set this equal to the length of either the array that was passed, or—if we're using a `model()` method—the length of `this.options.min`.

Kicking off our loop, we assign the `value` variable (what we'll be inserting) equal to either the current item being iterated in the passed array, or, by calling the model method directly. For good measure, we pass the current iteration to the model method just in case we want to make use of it later (why not, eh?!). Last but not least, we make The Big Decision™. If we're attempting to insert into the `users` collection, we make a call to `this.createUser()` (our next and final task). Otherwise—assuming a vanilla collection—we simply call `this.collection.insert( value )`!

Almost done! So close! Don't give up! Don't turn back now! The only way out is through! Okay, drama club. Pipe down. Last step, let's take a peek at `this.createUser()`.

### Handling user creation
Technically speaking, we don't want to insert users directly into the `Meteor.users` collection. Instead, we want to defer to Meteor's `Accounts.createUser()` method to ensure that the user is properly created. Let's hop down to our own `this.createUser()` method and see how we're utilizing it.

<p class="block-header">/server/seed.js</p>

```javascript
class Seeder {
  constructor( collection, options ) {
    [...]
  }

  [...]

  sow( data ) {
    [...]
  }

  [...]

  createUser( user ) {
    let isExistingUser = Meteor.users.findOne( { 'emails.address': user.email } );
    if ( !isExistingUser ) {
      let userId = Accounts.createUser({
        email: user.email,
        password: user.password,
        profile: user.profile || {}
      });

      if ( user.roles && Roles !== 'undefined' ) {
        Roles.addUsersToRoles( userId, user.roles );
      }
    }
  }
}

[...]
```

Inside, we take in our user and do a quick check to make sure that we haven't already created this user before. If we _haven't_, we go ahead and make a call to `Accounts.createUser()` passing in an `email` and `password` field. Optionally, if we have a `profile` value for our user we set it, otherwise, we just set `profile` to an empty object. Notice that here, we're simply setting our call to `Accounts.createUser()` to the variable `userId`. This is because on the server, this method can only be called synchronously—no callback—and it returns a `userId` for the newly created user when called.

Making use of this nifty feature, we do a quick check to see if our user has a `roles` property defined and that the roles package's `Roles` global is not undefined. If all is well, we take in the `userId` and the passed set of roles and apply them to the new user with `Roles.addUsersToRoles()`.

Done! At this point, our class is finished. We've succesfully wired up our seeder to support both an array of data _and_ creating copies of a model. This is great. We could stop here, but to make things clear let's finish out the recipe by preparing a few examples for Henrik and Sofia!

## Using our seeder
Great work! At this point our seeder is all ready to go. To wrap this up in a neat little bow for Henrik and Sofia, let's set up a few demos. Speeding things up, we've already done quite a bit of work wiring up a snazzy little template to two collections on the client: `users` and `products`. Our focus now will be on wiring up two methods for seeding the collection as well as clearing it out (to show the effect of the seeding).

<p class="block-header">/both/methods/seed.js</p>

```javascript
Meteor.methods({
  seedCollection( collection ) {
    check( collection, String );
    let isUsers = collection === 'Users';

    if ( isUsers ) {
      Seed( 'Users', {
        data: [
          {
            email: 'henrik@brainbots.fm',
            password: 'password',
            profile: {
              name: { first: 'Henrik', last: 'Müller' }
            },
            roles: [ 'admin' ]
          },
          {
            email: 'sofia@brainbots.fm',
            password: 'password',
            profile: {
              name: { first: 'Sofia', last: 'Carmichael' }
            },
            roles: [ 'admin' ]
          }
        ]
      });
    } else {
      Seed( 'Products', {
        min: 5,
        environments: [ 'development', 'staging', 'production' ],
        model( index ) {
          return {
            name: faker.commerce.product(),
            price: faker.commerce.price()
          };
        }
      });
    }
  },
  clearCollection( collection ) {
    check( collection, String );
    let isUsers = collection === 'Users';

    if ( isUsers ) {
      Meteor.users.remove({});
    } else {
      global[ collection ].remove({});
    }
  }
});
```

To call this method, we have a set of buttons for each demo on the client. One button is designed to _seed_ a collection, the other is designed to empty it. Here's what the markup looks like:

<p class="block-header">/client/templates/public/index.html</p>

```markup
<!-- Users demo -->
<button data-collection="Users" class="btn btn-success">Seed Users</button>
<button data-collection="Users" class="btn btn-danger">Clear Users</button>

<!-- Products demo -->
<button data-collection="Products" class="btn btn-success">Seed Products</button>
<button data-collection="Products" class="btn btn-danger">Clear Products</button>
```

Tied to each button is an event (one event for seeding, one for clearing) that grabs the `data-collection` value of the button. This is passed directly to the server by calling the appropriate method, either `seedCollection` or `clearCollection`. Our `clearCollection` method is pretty simple. It's designed to simply wipe out the appropriate collection entirely. Up above, we can see this happening with a simple check to see if we're deleting from the users collection or not—this should look familiar. If we are, we wipe it, otherwise, we make use of that nifty `global` object and just pass in the collection sent to us from the client!

For seeding, we've set up two demos specific to our collections `users` and `products`. For users, we've pulled in our example from the beginning of the recipe which simply creates two users using the data array method of our `Seeder` class. For the `products` collection, we pull in our `model()` method of seeding, specifying an object to return. Since we're just playing around, we've added two fields and assigned their values to be the result of calls to `faker.commerce.product()` and `faker.commerce.price()`. These come to us via the `digilord:faker` package that we installed earlier (and by proxy the [Faker library](https://github.com/Marak/faker.js)).

<figure>
  <img src="https://tmc-post-content.s3.amazonaws.com/seeder-demo.gif" alt="Seeding our collections via the demo.">
  <figcaption>Seeding our collections via the demo.</figcaption>
</figure>

Using these, each time our `model()` method is called—remember, this happens in the loop of our `sow()` method—these will be called, generating a new random product name and price. Pretty neat! This is all we need to do, too. If we boot up our app (make sure you're doing this from the cloned [source of the app](https://github.com/themeteorchef/writing-a-database-seeder)), we should see some options to seed and clear our collections.

Henrik and Sofia are going to love this, let's ship it!

## Wrap up & summary
In this recipe, we learned how to build a database seeder to help us generate test data. We learned how to write an ES2015 class, how to support multiple types of functionality, and how to use that class. We also took a look at using the [Faker](https://github.com/Marak/faker.js) library to help us with generating random data and how to add control flow based on the current application environment.
