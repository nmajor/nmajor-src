---
title: Simple Object Storage in Redis (Node.js)
layout: post
date: 2018-09-20 00:00:00 +0000
hero: ''
tags:
- node
- redis
- javascript

---
[Redis](https://redis.io/ "https://redis.io/") is a simple key value store and is highly optimized for fast reads and writes. I found myself in a situation where I wanted to offload some app task logging from our document store (mongoDB) to redis.

There are a few important things to consider when making this kind of change.

**First** it was important for our use case to have some basic kind of indexing and sorting based on when the document was created.

**Second**, since we do a lot of logging, we needed to make sure that all documents clean up after themselves and expire nicely so we don't run out of space in redis, and we don't really care about keeping these logs indefinitely.

**Third**, our logs are javascript objects, so we needed some kind of customizable serialization/deserialization.

**Fourth**, I wanted to have a nice abstraction with predictable model-like behavior like `findOne`, `create`, `update`, and `fromIndex`.

So here is a basic solution I came up with.

The plan:

* **Schema** - Add a schema for `redisModels`
* **Finding, Creating and Updating** - Use [redis hashes]() to store our objects and have functions for adding, finding, and updating.
* **Indexing and Sorting** - Use [redis lists](https://redis.io/topics/data-types#lists "https://redis.io/topics/data-types#lists") (aka ranges) to maintain a sorted list of object keys.

I'll be using the main [redis node](https://www.npmjs.com/package/redis "https://www.npmjs.com/package/redis") library for this.

### Schema

Here is what our basic schema will look like:

    const logSchema = {
      namespace: 'logs',
      indexes: [
        {
          getName: () => 'createdAt',
          shouldIndex: () => true,
          addNonTenantIndex: () => true,
          getValue: data => new Date(data.createdAt).getTime(),
        },
      ],
      attributes: {
        source: { kind: 'string' },
        user: { kind: 'object' },
        body: { kind: 'object' },
        createdAt: { kind: 'time' },
      },
    };

As you can see we set information about our `attributes`, `indexes` and a `namespace` that we will use for our redis keys. The `namespace` is mostly useful if you plan on storing multiple kinds of objects in redis.

### Creating

We'll start by creating a new class and constructor inside of a new file called `lib/redis.js`. I know this is a bunch of code to dump in all at once, but I'll break it down afterwards:

    import shortid from 'shortid';
    import _ from 'lodash';
    
    function getRedisClient() {
      // IMPLEMENT YOUR OWN METHOD OF GETTING THE REDIS CLIENT,
    }
    
    function expSeconds(days = 30) {
      const secondsPerDay = 86400;
      return days * secondsPerDay;
    }
    
    function buildId() { return shortid.generate(); }
    
    const stringify = {
      string: data => data,
      number: data => data.toString(),
      date: data => new Date(data).toISOString(),
      time: data => new Date(data).toISOString(),
      object: data => JSON.stringify(data),
    };
    
    export default class RedisModel {
      constructor(schema) {
        this.props = {
          schema,
          client: getRedisClient(),
        };
      }
    
      _buildHashValues(data) {
        const { schema: { attributes } } = this.props;
        const hashValues = [];
    
        _.each(data, (value, key) => {
          const type = (attributes[key] || {}).kind;
          const stringValue = stringify[type] ? stringify[type](value) : undefined;
          if (!type || !stringValue) return null;
    
          return hashValues.push(key, stringValue);
        });
    
        return _.isEmpty(hashValues) ? undefined : hashValues;
      }
    
      _buildRedisKey(id) {
        const { schema } = this.props;
        return `${schema.namespace}:${id}`;
      }
    
      create(data) {
        const { client } = this.props;
    
        return new Promise((resolve, reject) => {
          const multi = client.multi();
          const id = buildId();
          const redisKey = this._buildRedisKey(id);
    
          // Handle the hash values
          const hashValues = this._buildHashValues(data);
          if (hashValues) {
            multi.hmset(redisKey, hashValues);
            multi.hmset(redisKey, 'EX', expSeconds());
            multi.exec((err) => {
              if (err) return reject(err);
    
              return resolve({ key: redisKey, _id: id, ...data });
            });
          } else {
            reject(new Error('Empty redis hash data'));
          }
        });
      }
    }

First of all you'll notice I'm using the [lodash]() for its useful helper methods, and [shortid](https://www.npmjs.com/package/shortid "https://www.npmjs.com/package/shortid") to help generate unique ids for our redis keys.

We are using the `multi` redis behavior.

    const multi = client.multi();
    // ...
    multi.hmset(redisKey, hashValues);
    multi.hmset(redisKey, 'EX', expSeconds());
    multi.exec((err) => {
    // ...

This allows us to do multiple calls in one redis transaction, and if one of the calls fails, the whole transaction will fail. This reduces the chances that we'll have stray data being stored in the database.

We're also using our schema namespace to build a unique redis key for our object.

    const redisKey = this._buildRedisKey(id);
    
      _buildRedisKey(id) {
        const { schema } = this.props;
        return `${schema.namespace}:${id}`;
      }

This isn't totally necessary, and really only useful if you are storing multiple types of objects in redis, but it does give us the ability to do basic fetching redis for all items with the namespace prefix later if we want.

You can see that we're using a [redis hash](https://redis.io/topics/data-types#hashes "https://redis.io/topics/data-types#hashes") to store all our object data.

    multi.hmset(redisKey, hashValues);

The [hmset](https://redis.io/commands/hmset "https://redis.io/commands/hmset") function takes an array of key value pairs. Redis hashes require the values to be strings, so our `_buildHashValues` function takes our data and converts it to an array of strings based on the key/value pairs and attribute types in the schema.

      _buildHashValues(data) {
        const { schema: { attributes } } = this.props;
        const hashValues = [];
    
        _.each(data, (value, key) => {
          const type = (attributes[key] || {}).kind;
          const stringValue = stringify[type] ? stringify[type](value) : undefined;
          if (!type || !stringValue) return null;
    
          return hashValues.push(key, stringValue);
        });
    
        return _.isEmpty(hashValues) ? undefined : hashValues;
      }

And it uses our `stringify` object as a map for how to convert the different attributes to strings based on the schema. This gives us some granular control over the serialization of each attribute of our objects.

And we're also setting an expiration for our keys so they dont last forever in our database.

    multi.hmset(redisKey, 'EX', expSeconds());

And that handles our ability to create new objects in our redis model.

### Updating

Our update function is very similar to our create:

    export default class RedisModel {
      // ...
      update(id, data) {
        const { client } = this.props;
    
        return new Promise((resolve, reject) => {
          const redisKey = this._buildRedisKey(id);
    
          // Handle the hash values
          const hashValues = this._buildHashValues(data);
          if (hashValues) {
            client.hmset(redisKey, hashValues, (err) => {
              if (err) return reject(err);
    
              return resolve();
            });
          } else resolve();
        });
      }
    }

Basically it uses the same `_buildRedisKey` function so we only have to deal with the id of the object. Then it basically just does the same `hmset` function. Redis will keep any attributes in the hash that are not overwritten.

### Finding

Here's our find method. This lets us find any object by id and deserialize all the values into their original datatypes based on the schema:

    const parsify = {
      string: data => data,
      number: data => parseInt(data, 10),
      date: data => new Date(data),
      time: data => new Date(data),
      object: data => JSON.parse(data),
    };
    
    export default class RedisModel {
      // ...
      _unpackHashValues(data) {
        const { schema: { attributes } } = this.props;
        const obj = {};
    
        _.each(data, (value, key) => {
          const type = (attributes[key] || {}).kind;
          if (!type) return null;
    
          obj[key] = parsify[type](value);
          return null;
        });
    
        return _.isEmpty(obj) ? undefined : obj;
      }
    
      findOne(id) {
        const { client } = this.props;
    
        return new Promise((resolve, reject) => {
          const redisKey = this._buildRedisKey(id);
          client.hgetall(redisKey, (err, data) => {
            if (err) return reject(err);
    
            return resolve({
              key: redisKey,
              _id: id,
              ...this._unpackHashValues(data),
            });
          });
        });
      }
    }

Here is where we use a `parsify` method to convert our attribute strings into their original data type.

And that handles our finding and deserialization.

### Indexing and Sorting

Here is the tricky part. Redis, as a key/value store doesnt have any real sense of ordering when it comes to its keys. But its important for our use case to be able to see the most recent logs, and maybe later be able to query date ranges of when the logs were created.

So what we're going to do is use the redis function `zadd` to add the redis key and timestamp for each object into a sorted redis list.

Remember from our schema, indexes look like this:

    {
      getName: () => 'createdAt',
      shouldIndex: () => true,
      getValue: data => new Date(data.createdAt).getTime(),
    }

`getName` is a function because I've found it useful to be able to create some indexes that are isolated in different ways. For example I've sometimes added indexes that are based on the ID of the user who created the log, this lets me easily get a list of all the user's most recent indexes. And in that case the `getName` looks like this `getName: data => ('user:'+data.user._id+':createdAt')`.
    

`shouldIndex` lets us not index any object we want, and `getValue` gives us the actual value of the index, so this actually lets us create sorted lists using any different attribute not just createdAt. Just remember that `getValue` has to return a number.

Now we modify our `create` function to include a `zadd` for each index in our schema:

    _buildIndexName(indexName) {
        const { schema } = this.props;
        return `${schema.namespace}:${indexName}`;
      }
    
      create(data) {
        const { schema, client } = this.props;
    
        return new Promise((resolve, reject) => {
          const multi = client.multi();
          const id = buildId();
          const redisKey = this._buildRedisKey(id);
    
          // Handle the hash values
          const hashValues = this._buildHashValues(data);
          if (hashValues) {
            multi.hmset(redisKey, hashValues);
            multi.hmset(redisKey, 'EX', expSeconds());
    
            // Handle indexes
            if (schema.indexes) {
              _.each(schema.indexes, (index) => {
                if (index.shouldIndex(data)) {
                  multi.zadd(this._buildIndexName(index.getName(data)), index.getValue(data), redisKey);
                }
              });
            }
    
            multi.exec((err) => {
              if (err) return reject(err);
    
              return resolve({ key: redisKey, _id: id, ...data });
            });
          } else {
            reject(new Error('Empty redis hash data'));
          }
        });
      }

This inserts the `redisKey` of each object into a sorted list based on the createdAt timestamp of the object.

Now we can easily retrieve the top objects from any index with a function like this:

    export default class RedisModel {
      // ...
      _clearOldIndexes(indexName) {
        const { client, schema } = this.props;
        return new Promise((resolve, reject) => {
          const now = new Date().getTime();
          const secondsPerDay = 86400;
          const args = [
            this._buildIndexName(schema, indexName),
            (now + (expSeconds() - secondsPerDay) * 1000),
            '-inf',
          ];
    
          client.zremrangebyscore(args, (err, results) => {
            if (err) return reject(err);
    
            return resolve(results);
          });
        });
      }
    
      _getIndexedIds(indexName, offset, limit) {
        const { client } = this.props;
    
        return this._clearOldIndexes(indexName)
          .then(() => new Promise((resolve, reject) => {
            const args = [indexName, '+inf', '-inf', 'LIMIT', offset || 0, limit || 20];
    
            client.zrevrangebyscore(args, (err, results) => {
              if (err) return reject(err);
    
              return resolve(results);
            });
          }));
      }
    
      fromIndex(indexName) {
        return this._getIndexedIds(indexName, undefined, 250)
          .then(results => Promise.all(results.map(result => this.findOne(result))));
      }
    }

Notice that inside `_getIndexedIds` we first `_clearOldIndexes`. This is because redis does not allow us to set expiration times for entries inside of a list so I get around this by first removing all entries that are expired before actually getting a list of all the entries in the list.

Then it gets a list of all the ids using the [zrangebyscore]() function. Then it converts each one into its full object using the `findOne` function that we already made.

### Usage

Now that we have our basic `RedisModel` class we can create new models by just exporting an instance with our schema. For example we could have a `models/log.js` file that looks like this:

    // models/log.js
    import RedisModel from '../lib/redis';
    
    const logSchema = {
      namespace: 'logs',
      indexes: [
        {
          getName: () => 'createdAt',
          shouldIndex: () => true,
          addNonTenantIndex: () => true,
          getValue: data => new Date(data.createdAt).getTime(),
        },
      ],
      attributes: {
        source: { kind: 'string' },
        user: { kind: 'object' },
        body: { kind: 'object' },
        createdAt: { kind: 'time' },
      },
    };
    
    export default new RedisModel(schema);

Then in other parts of our program we can do things like this:

    import Log from '../models/log';
    
    // Get a list of logs
    const logList = Log.fromIndex('createdAt');
    
    // Find a specific log
    Log.findOne('123456');
    
    // Create a new log
    Log.create({
      source: 'Web',
      body: 'An error happened',
      user: { email: 'joe@cool.com',  },
      createdAt: new Date(),
    });

And based on how we built it, we think about our individual objects in terms of `ids` and not keys.

### Conclusion

You can see a [gist with the full RedisModel class here](https://gist.github.com/nmajor/e772d0dd166c8c3bbffb2bee00faa8a2 "https://gist.github.com/nmajor/e772d0dd166c8c3bbffb2bee00faa8a2"). Keep in mind this is mostly just to give a possible starting point. Its possible there are some bugs in here since I had to water down my implementation a bit to keep this post concise.

As always, please let me know if you see any issues with the code, or possible problems with my implementation or design choices. I'm always learning.

Thanks for reading!