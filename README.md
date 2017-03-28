<big><h1 align="center">spikenail</h1></big>

<big><h1 align="center">WORK IN PROGRESS</h1></big>

<p align="center">
  <a href="https://npmjs.org/package/spikenail">
    <img src="https://img.shields.io/npm/v/spikenail.svg?style=flat-square"
         alt="NPM Version">
  </a>

  <a href="https://coveralls.io/r/spikenail/spikenail">
    <img src="https://img.shields.io/coveralls/spikenail/spikenail.svg?style=flat-square"
         alt="Coverage Status">
  </a>

  <a href="https://travis-ci.org/spikenail/spikenail">
    <img src="https://img.shields.io/travis/spikenail/spikenail.svg?style=flat-square"
         alt="Build Status">
  </a>

  <a href="https://npmjs.org/package/spikenail">
    <img src="http://img.shields.io/npm/dm/spikenail.svg?style=flat-square"
         alt="Downloads">
  </a>

  <a href="https://david-dm.org/spikenail/spikenail.svg">
    <img src="https://david-dm.org/spikenail/spikenail.svg?style=flat-square"
         alt="Dependency Status">
  </a>

  <a href="https://github.com/spikenail/spikenail/blob/master/LICENSE">
    <img src="https://img.shields.io/npm/l/spikenail.svg?style=flat-square"
         alt="License">
  </a>
</p>

Spikenail is an open-source Node.js ES7 framework that allows you to build GraphQL API with little or no coding.

## Features

Full support of ES7 features

Native GraphQL support

Relay compatible API

## Install

```
npm install -g generator-spikenail
yo spikenail
```

## Core concepts

Ability to build an API just by configuring it is the main idea of spikenail.
That configuration might include relations, access control, validations and everything else we need.

At the same time we should provide enough flexibility by allowing to adjust or override every action spikenail does.
From this point of view, spikenail provides an architecture and default implementation of it.

The configuration mentioned above stored in models.

Example model `models/Item.js`:

```js
import { MongoDBModel } from 'spikenail';

class Item extends MongoDBModel {

  /**
   * Example of custom method
   */
  customMethod() {
    // Access underlying mongoose model
    return this.model.find({ 'category': 'test' }).limit(10);
  }
}

export default new Item({
  name: 'item',
  properties: {
    id: {
      type: 'id'
    },
    name: {
      type: String
    },
    description: {
      type: String
    },
    position: {
      type: Number
    },
    token: {
      type: String
    },
    virtualField: {
      virtual: true,
      // Ensure that dependent fields will be queried from the database
      dependsOn: ['position'],
      type: String
    },
    userId: {
      type: 'id'
    },
    // Relations
    subItems: {
      relation: 'hasMany',
      ref: 'subItem',
      foreignKey: 'itemId'
    },
    user: {
      relation: 'belongsTo',
      ref: 'user',
      foreignKey: 'userId'
    }
  },
  // Custom resolvers
  resolvers: {
    description: async function(_, args) {
      // It is possible to do some async actions here
      let asyncActionResult = await someAsyncAction();
      return asyncActionResult ? _.description : null;
    },
    virtualField: (_, args) => {
      return 'justCustomModification' + _.position
    }
  },
  validations: [{
    field: 'name',
    assert: 'required'
  }, {
    field: 'name',
    assert: 'maxLength',
    max: 100
  }, {
    field: 'description',
    assert: 'required'
  }],
  acls: [{
    allow: false,
    properties: ['token'],
    actions: ['*']
  }, {
    allow: true,
    properties: ['token'],
    actions: [ACTION_CREATE]
  }]
});
```

### CRUD

In spikenail every CRUD action is a set of middlewares.
These middlewares are not the request middlewares and exists separately.

Default middlewares are:

* Access control middleware
* Validation middleware
* Before action
* Process action
* After action

The whole chain could be changed in any way.

Example of how "Before action" middleware could be overriden:

In the model class:

```js

  async beforeCreate(result, next, opts, input, ctx) {
    let checkResult = await someAsyncCall();

    if (checkResult) {
        return next();
    }

    result.errors = [{
        message: 'Custom error',
        code: '40321'
    }];
  }

```

## Configuration

Configuration files are stored under config folder

### Data sources

Currently, only MongoDB supported.

It is recommended to store all configuration using environment variables

Example of `config/sources.js`

```js
export default {
  'default': {
    adapter: 'mongo',
    connectionString: process.env.SPIKENAIL_MONGO_CONNECTION_STRING
  }
}
```

## GraphQL API

### Queries

#### node

```js
node(id: ID!): Node
```

https://facebook.github.io/relay/docs/graphql-object-identification.html#content

#### viewer

Root field

```
viewer: viewer

type viewer implements Node {
  id: ID!
  user: User,
  allXs(): viewer_XConnection
}
```


#### Query all items of a specific model

For `Article` model:

```
query {
    viewer {
        allArticles() {
            id, title, text
        }
    }
}
```


#### Query single item

Query a specific article by unique field:

```
query {
    getArticle(id: "article-id-1") {
        id, title, text
    }
}
```

#### Relation queries

#### Filtering queries

#### Mutations

## Creating a Model

### Relations

#### hasMany relation

`models/Book.js`

```js
properties: {
    authors: {
      relation: 'hasMany',
      ref: 'author',
      foreignKey: 'bookId'
    }
}
```

##### Custom hasMany condition

```js
 getConditions: function(_) {
    return { otherModelField: _.name }
 }
```

#### MongoDBModel

Underlying model is a [mongoose](http://mongoosejs.com/) model. You can access it through `this.model`

##### Changing collection name

```js
providerOptions: {
    collection: 'customName'
}
```


### Adding custom method

## Koa middlewares

Spikenail is based on koa2.

### Adding your own koa middlewares

## Authentication

## ACL

### Introduction

```
acls: [{
    allow: false,
    roles: ['*'],
    actions: ['*']
}, {
    allow: true,
    roles: ['*'],
    actions: ['*'],
    scope: function() {
        return { isPublic: true }
    }
}
```

ACL rules are specified under `acls` property of model schema
Rules applies one by one the in natural order priority.

Rules notation could be simplified and above rules might be written as

```

```

### Rule structure

#### allow

Each rule must have `allow` property defined. Allow is boolean value
that indicates if rule allows something or disallows.

Example:
```
allow: true
```

#### properties (optional)
`properties` is an array of properties of model that rule should apply to.
Omit or use * sign to apply to all rules

#### actions (optional)

Specify what actions rule should be applied to
There are 4 types of actions:2

* create
* update
* remove
* read

Omit this property or use * sign to apply to all actions

Example:

```
properties: ['create', 'update']
```

#### scope

Scope is a mongodb condition. If document match a scope only then rule will be applied.

e.g. `{ isPublic: true }`
The rule will be applied only to documents that have `isPublic` property equals `true`

Scope might be defined as function

`
scope: function(ctx) {
        return { isPublic: true }
}
`

This way it will have access to ctx

#### roles

`roles` is an array of roles that rules apply to.

Example

```
roles: ["anonymous", "member"]
```

Roles might be static or dynamic

##### Static roles

Static roles is the roles that not depends on particular document or dataset.
They calculated once per request for current user.

Built-in roles are

* anonymous
* user

###### Adding your own static roles

Override getStaticRoles function of the model

##### Dynamic roles

Dynamic roles are calculated for each particular document.
For example role `owner` means that `currentUser.id === fetchedDocument.id`

Built-in roles are

* owner

###### Defining dynamic roles

Dynamic roles are defined using `roles` object of model schema

For example we want to share some object with other users,
and put their username into `members` array of the document.

Then we can define role `member` in model schema:

```
roles: {
     member: {
       cond: function(ctx) {
         return { "members.userId": ctx.currentUser }
       }
     }
   }
```

In roles property of acl rule:

```
roles: ['member']
```


#### Access based on another model

In some cases we want to apply rule only if another model satisfies the condition
There are two ways to do this:

##### checkRelation

Example:

`Article.js` model has defined belongsTo relation

```
blog: {
    relation: "belongsTo"
}
```

We want allow for `user` to only read a article only if he can read the blog article belongs to:
```
acls: [{
    allow: false
}, {
    allow: true,
    roles: ["user"],
    actions: ["read"],
    checkRelation: "blog"
    checkAction: "read"
}]
```

checkAction is an optional property. It will match current invoked action by default.

##### test

```
test: "blog"
```

The access rules for the model will completely depends on rules for the relation `blog`.
If role X can do action Y on blog, same role can do same action on current model.


## License

MIT © [Igor Lesnenko](http://github.com/spikenail)

[npm-url]: https://npmjs.org/package/spikenail
[npm-image]: https://img.shields.io/npm/v/spikenail.svg?style=flat-square

[travis-url]: https://travis-ci.org/spikenail/spikenail
[travis-image]: https://img.shields.io/travis/spikenail/spikenail.svg?style=flat-square

[coveralls-url]: https://coveralls.io/r/spikenail/spikenail
[coveralls-image]: https://img.shields.io/coveralls/spikenail/spikenail.svg?style=flat-square

[depstat-url]: https://david-dm.org/spikenail/spikenail
[depstat-image]: https://david-dm.org/spikenail/spikenail.svg?style=flat-square

[download-badge]: http://img.shields.io/npm/dm/spikenail.svg?style=flat-square
