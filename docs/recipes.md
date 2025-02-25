# Recipes

## Transactions

Because Papr model methods are simple wrappers around the official MongoDB driver methods,
they can be easily used with [MongoDB transactions](https://www.mongodb.com/docs/manual/core/transactions/).

```ts
import { MongoClient } from 'mongodb';
import Papr, { schema, types } from 'papr';

const papr = new Papr();

const client = await MongoClient.connect('mongodb://localhost:27017');

papr.initialize(connection.db('test'));

const User = papr.model(
  'users',
  schema({
    age: types.number(),
    firstName: types.string({ required: true }),
    lastName: types.string({ required: true }),
  })
);

await papr.updateSchemas();

const session = client.startSession();

try {
  await session.withTransaction(async () => {
    await User.insertOne(
      {
        firstName: 'John',
        lastName: 'Wick',
      },
      { session }
    );
    await User.insertOne(
      {
        firstName: 'John',
        lastName: 'Doe',
      },
      { session }
    );
  });
} finally {
  await session.endSession();
}
```

## Mongoose Migration

Migrating your Mongoose models to Papr is straightforward, but it's important to remember that Papr is not an ORM, so some of the functionality baked into your Mongoose models will need to be handled separately. This includes custom instance or static methods, query helpers, and index definitions.

### Timestamps

Timestamp support with Papr is exactly the same as Mongoose. You can define custom timestamp property names in the config options the same as you would with Mongoose. An example of this can be seen below.

```js
import mongoose from 'mongoose';
const { Schema } = mongoose;

const exampleSchema = new Schema(
  {
    name: { type: String },
  },
  {
    timestamps: true,
  }
);
```

```ts
import { schema, types } from 'papr';

const exampleSchema = schema(
  {
    name: types.string(),
  },
  {
    timestamps: true,
  }
);
```

### Default Values

Papr does not support dynamic default values, but static default values can be used. Unlike Mongoose where default values are defined in the individual property options, Papr defines defaults in the schema options. An example of this can be seen below.

```js
import mongoose from 'mongoose';
const { Schema } = mongoose;

const exampleSchema = new Schema({
  hidden: { type: Boolean, default: false, required: true },
});
```

```ts
import { schema, types } from 'papr';

const exampleSchema = schema(
  {
    switch: types.boolean({ required: true }),
  },
  {
    defaults: {
      switch: false,
    },
  }
);
```

### Version Key

Mongoose automatically adds a `versionKey` to all of your schemas - you will need to either remove that value from your collections or include the matching key in your Papr schema when migrating. The default value for this property is `__v`, but may be changed in your Mongoose schema options. An example of this can be seen below.

```js
import mongoose from 'mongoose';
const { Schema } = mongoose;

const exampleSchema = new Schema({
  name: { type: String },
});
```

```ts
import { schema, types } from 'papr';

const exampleSchema = schema({
  __v: types.number(),
  name: types.string(),
});
```

### Complete Example

This is an example of a complete schema migration from Mongoose to Papr with all of the above considerations taken into account.

```js
import mongoose from 'mongoose';
const { Schema } = mongoose;

const postSchema = new Schema(
  {
    title: { type: String, required: true },
    author: { type: String, required: true },
    body: { type: String, required: true },
    comments: [{ body: String, date: Date }],
    hidden: { type: Boolean, default: false, required: true },
    meta: {
      votes: Number,
      favs: Number,
    },
    deletedAt: { type: Date },
  },
  { timestamps: true }
);
```

```ts
import { schema, types } from 'papr';

const postSchema = schema(
  {
    __v: types.number(),
    title: types.string({ required: true }),
    author: types.string({ required: true }),
    body: types.string({ required: true }),
    comments: types.array(
      types.object({
        body: types.string(),
        date: types.date(),
      })
    ),
    hidden: types.boolean({ required: true }),
    meta: types.object({
      votes: types.number(),
      favs: types.number(),
    }),
    deletedAt: types.date(),
  },
  {
    timestamps: true,
    defaults: {
      hidden: false,
    },
  }
);
```
