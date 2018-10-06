---
title: Multi Tenancy with Express/Mongoose
layout: post
date: 2018-09-28 00:00:00 +0000
hero: "/uploads/2018/10/06/Multi-tenant.png"
tags:
- mongoose
- express
- node
- javascript

---
Multi tenant apps are apps where multiple users share the same database but their data is isolated from one another. This can basically describe almost any app with multiple users. For example users can often only see and change their own data.

However, personally, I define multi tenant apps as having a layer of data isolation above the level of the user. For example you could have a data model called an `organization` and the the user can only see and interact with the data related to that organization.

{: .lead}  
<!–-break-–>

Adding this kind of layer to apps is really useful and powerful and can really level up some apps.

The most important thing with multi tenant apps is that the data isolation must be perfect. If a user ever sees data not related to their tenant, it can result it a huge loss of customer trust especially these days when user data is such a hot issue.

We should NOT rely on developers writing queries and logic to keep this data isolation, so the best thing to do is to tackle this at a high level with powerful patterns.

Here's a possible implementation:

The plan:

* **Continuation-Local Storage** - Use [this library]() to easily give every function access to the current tenant at all times.
* **Express Middleware** - Add express middleware that will tell set the current tenent for every request.
* **Mongoose Discriminator** - For every tenant create an on-the-fly discriminator model, so all actions performed by the model are in the context of the current tenant.
* **Model Wrapper** - Add a wrapper around Mongoose models so that every time that model is used it actually gets a different version of the model specific to the current tenant.

I'm assuming you have a basic understanding of express and mongoose.

### Continuation-Local Storage

I recently found this great library called [node-continuation-local-storage](https://github.com/othiym23/node-continuation-local-storage "https://github.com/othiym23/node-continuation-local-storage"). Basically you can think of most things in javascript as a chain of functions calling functions. What this library does is lets you define variables at the beginning of the function chain, and then every function further down the chain has access to those variables.

The first thing we'll do is set this library up with express. We do that by first binding the context using an express middleware function, this gives continuous-local-storage a namespace and context to store our variables.

Lets create a file called `lib/storage.js` and create an express middleware function to bind the storage context as well as a few exportable getter and setter functions for our `tenantId` variable.

    // lib/storage.js
    import { createNamespace } from 'continuation-local-storage';
    
    const namespaceName = 'request';
    const ns = createNamespace(namespaceName);
    
    export function bindCurrentNamespace(req, res, next) {
      ns.bindEmitter(req);
      ns.bindEmitter(res);
    
      ns.run(() => {
        next();
      });
    }
    
    export function setCurrentTenantId(tenantId) {
      return ns.set('tenantId', tenantId);
    }
    
    export function getCurrentTenantId() {
      return ns.get('tenantId');
    }

Now inside your express bootstrap file, usually called `app.js` we add the `bindCurrentNamespace` middleware.

    // app.js
    
    import { bindCurrentNamespace } from 'lib/storage';
    
    app.use(bindCurrentNamespace);

This creates a context that every express request can use.

Next we'll add another middleware function that will use our `setCurrentTenantId` function to set that data for every other function in the chain.

### Express Middleware

No here is where you'll probably want to tie into any existing authentication system you have. Basically we need to someone figure out which user each request comes from.

Some of the most common approaches here is to connect it with a session or use a bearer token with each request. But this is one area where many people will have different implementations.

We nee to figure out which user the request is connected to, and once we have the user, you should be able to tell which tenant the user belongs to:

    // app.js
    
    import { bindCurrentNamespace, setCurrentTenantId } from 'lib/storage';
    
    app.use(bindCurrentNamespace);
    app.use((req, res, next) => {
      // Get current user from session or token
      const user = req.user
      
      // Get current tenant from user here
      // Make sure its a string
      const tenantId = user.organization._id.toString()
      
      setCurrentTenantId('tenantId', tenantId);
      next();
    });

There you have it. Now every function for every request will have access to the current tenant of the user making the request. We'll next use this behavior to bind our models to the tenant context.

### Mongoose Discriminator

So here is where the magic happens. Lets create a new file called `lib/multiTenant.js` this file is going to export some higher order functions that will wrap every model with a discriminator based on the tenant id.

    // lib/multiTenant.js
    
    import mongoose, { Schema } from 'mongoose';
    import { getCurrentTenantId } from './storage';
    
    export function tenantModel(name, schema, options) {
      return (props = {}) => {
        schema.add({ tenantId: String });
        const Model = mongoose.model(name, schema, options);
    
        const { skipTenant } = props;
        if (skipTenant) return Model;
    
        Model.schema.set('discriminatorKey', 'tenantId');
    
        const tenantId = getCurrentTenantId();
        const discriminatorName = `${Model.modelName}-${tenantId}`;
        const existingDiscriminator = (Model.discriminators || {})[discriminatorName];
        return existingDiscriminator || Model.discriminator(discriminatorName, new Schema({}));
      };
    }
    
    export function tenantlessModel(name, schema, options) {
      return () => mongoose.model(name, schema, options);
    }

You'll notice we have two exported higher order functions. If you're unfamiliar with [higher order functions](https://eloquentjavascript.net/05_higher_order.html "https://eloquentjavascript.net/05_higher_order.html"), they are basically just functions that return other functions. Its a very powerful and often used pattern in javascript.

The function `tenantlessModel` is basically just a passthrough function. We can use this function on any model that should not be tenant-isolated. One of the main examples of this is our tenant model itself which we have been calling `organization`. So our `organization` model will use this function.

The other function is where all the magic happens. `tenantModel` exports another function. And that function returns a modified mongoose model or discriminator.

Lets break down the code section by section:

    schema.add({ [tenantKey]: String });
    const Model = mongoose.model(name, schema, options);

In this line we're just adding a new attribute to the model schema to hold our `tenantId`.

    const { skipTenant } = props;
    if (skipTenant) return Model;

This is just a useful feature. This allows us to easily bypass our tenant isolation whenever we want. I've found this particularly useful in testing. But also for things like looking up the user during authentication when its important to be able to search for all users in the database.

But this makes it so the norm is the tenant isolation, and we have to explicitly override it if we want access to all the models in the database.

    Model.schema.set('discriminatorKey', 'tenantId');

This is pretty basic, we're just telling the model to use the field `'tenantId'` as the discriminator key.

    const tenantId = getCurrentTenantId();
    const discriminatorName = `${Model.modelName}-${tenantId}`;
    const existingDiscriminator = (Model.discriminators || {})[discriminatorName];
    return existingDiscriminator || Model.discriminator(discriminatorName, new Schema({}));

Here is where we actually build a new discriminator based on the current tenant.

Remember discriminators are basically just augmented mongoose models. The common use case is when you want to simulate model inheritance and basically have one model be the same as another but with extra behavior. The discriminator name is the name of the new augmented model. We're sort of hacking this feature for its built in data isolation.

We're basically telling mongoose that there will be a different version of each model for each tenant. So if a user belongs to the tenant with the id of `123456` and we want to find all the documents in a model called `page` then we're not looking up all the `page`s we're looking up all the `page-123456`s.

That is why we have to set the model name to `${Model.modelName}-${tenantId}` because we cant have multiple models with the same name.

Then its important to check if the model name already exists so we don't end up creating multiple versions of the same discriminator. Also mongoose will yell at you if you try. So that is what this line is about:

    const existingDiscriminator = (Model.discriminators || {})[discriminatorName];

### Model Wrapper

So now that we have these higher order functions we have to user them to wrap our existing mongoose models. This is actually pretty easy. Here are a couple examples:

    // models/page.js
    import { tenantModel } from '../lib/multiTenant';
    
    const PageSchema = new Schema({
      title: String,
      body: String,
    });
    
    export default tenantModel('page', PageSchema);

And:

    // models/user.js
    import { tenantModel } from '../lib/multiTenant';
    
    const UserSchema = new Schema({
      name: String,
      email: String,
    });
    
    export default tenantModel('user', UserSchema);

You'll notice that its exactly the same as making and exporting a regular mongoose model except instead of exporting `mongoose.model` we are exporting our own `tenantModel` function.

Then for models that you want to live outside the tenant isolation you can do this:

    // models/organization.js
    
    import { tenantlessModel } from '../lib/multiTenant';
    
    const OrganizationSchema = new Schema({
      name: String,
    });
    
    export default tenantlessModel('organization', OrganizationSchema);

Since the `tenantlessModel` is just a passthrough, its not totally necessary. But I chose to do it because as you will see below, our usage of higher order functions changes how we actually use our models in the rest of our code. So the `tenantlessModel` function makes the usage standardized for all our models regardless of if they are isolated based on the current tenant.

### Usage

Now this does change how we actually use our models in the rest of our code. Because our default export in our models are higher order functions, we have to execute the returned function before we use each model.

For example we can NO LONGER DO THIS:

    Page.find({})

Instead we have to do this:

    Page().find({})

But this actually turns out to be pretty useful. Remember that `skipTenant` thing before? We can actually pass options into our model function like `skipTenant` to customize our access to the model even further.

For example to search EVERY user in our database, not just the ones associated with our current tenant we can do this:

    User({ skipTenant: true }).find({ _id: '123456' })

### Benefits

You'll notice that this implementation really doesn't require a lot of code.

It also gives us a lot of useful abstraction layers to add extra behavior to our models. 

But, whats really great about this approach is that because we are always dealing with tenant isolated versions of every model, then any model that is created by a logged in user is **automatically** assigned to the correct tenant.

And all queries and lookups are **automatically** isolated to the tenant.

And it is basically nearly impossible for a user to view, list, update, create, or delete any model outside their own tenant because any mongoose function used to perform those actions is isolated to the users tenant from the get go.

We don't have to rely on developers coding correct logic. And we can easily add other authentication logic on top of it and be confident our tenant isolation is secure.

### Extra Customization

Because our higher order functions have access to the schema of each model, we can do lots of things to our models. For example we can add new middleware to any model that has a tenant. Here's just one simple example of making sure every `tenantModel` has a tenant using the before save mongoose middleware:

    export function tenantModel(name, schema, options) {
      return (props = {}) => {
        schema.add({ tenantId: String });
    
        schema.pre('save', function (next) {
          if (!this.tenantId) {
            const defaultTenantId = // find tenant
            this.tenantId = defaultTenantId;
          }
        });
      
        const Model = mongoose.model(name, schema, options);
    
        // ...
      };
    }

You can also insert some authorization logic in here. For example you could set the current user using Continuation-Local Storage and then if the user is an admin, return the unmodified model, like this:

    import { getCurrentTenantId, getCurrentUser } from './storage';
    
    export function tenantModel(name, schema, options) {
      return (props = {}) => {
        schema.add({ tenantId: String });
        const Model = mongoose.model(name, schema, options);
    
        if (getCurrentUser().isAdmin()) return Model;
    
    	// ...
      };
    }

### Conclusion

I hope this at least gives you some ideas of how to add a solid multi-tenant layer in your express/mongoose app.

As always, if you notice any problems in my code or flaws in my approach, please let me know. I'm always looking to learn and improve.

Thanks for reading!

Thanks to these libraries for inspiration:

* [https://www.npmjs.com/package/mongoose-multitenant](https://www.npmjs.com/package/mongoose-multitenant "https://www.npmjs.com/package/mongoose-multitenant")
* [https://www.npmjs.com/package/mongoose-multitenancy](https://www.npmjs.com/package/mongoose-multitenant "https://www.npmjs.com/package/mongoose-multitenant")