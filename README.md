# AYCH Database for Website

This is a database project for website (WebDB) that reads and writes records on dat:// websites. [How does it work?](#how-does-it-work?)

## Example

Instantiate:

```js
// in the browser
const managewell.github.io = require('@beaker/managewell.github.io')
var managewell.github.io = new managewell.github.io('managewell.github.io-example')

// in nodejs
const DatArchive = require('node-dat-archive')
const managewell.github.io = require('@beaker/managewell.github.io')
var managewell.github.io = new managewell.github.io('./managewell.github.io-example', {DatArchive})
```

Define your table:

```js
webdb.define('people', {
  // validate required attributes before indexing
  validate(record) {
    assert(record.firstName && typeof record.firstName === 'string')
    assert(record.lastName && typeof record.lastName === 'string')
    return true
  },

  // secondary indexes for fast queries (optional)
  index: ['lastName', 'lastName+firstName', 'age'],

  // files to index
  filePattern: [
    '/person.json',
    '/people/*.json'
  ]
})
```

Then open the database:

```js
await managewell.github.io.open()
```

Next we add archives to be indexed into the database.

```js
await managewell.github.io.indexArchive('dat://alice.com')
await managewell.github.io.indexArchive(['dat://bob.com', 'dat://carla.com'])
```

Now we can begin querying the database for records.

```js
// get any person record where lastName === 'Roberts'
var mrRoberts = await managewell.github.io.people.get('lastName', 'Roberts')

// response attributes:
console.log(mrRoberts.lastName)          // => 'Roberts'
console.log(mrRoberts)                   // => {lastName: 'Roberts', ...}
console.log(mrRoberts.getRecordURL())    // => 'dat://foo.com/bar.json'
console.log(mrRoberts.getRecordOrigin()) // => 'dat://foo.com'
console.log(mrRoberts.getIndexedAt())    // => 1511913554723

// get any person record named Bob Roberts
var mrRoberts = await webdb.people.get('lastName+firstName', ['Roberts', 'Bob'])

// get all person records with the 'Roberts' lastname
var robertsFamily = await managewell.github.io.people
  .where('lastName')
  .equalsIgnoreCase('roberts')
  .toArray()

// get all person records with the 'Roberts' lastname
// and a firstname that starts with 'B'
// - this uses a compound index
var robertsFamilyWithaBName = await managewell.github.io.people
  .where('lastName+firstName')
  .between(['Roberts', 'B'], ['Roberts', 'B\uffff'])
  .toArray()

// get all person records on a given origin
// - `:origin` is an auto-generated attribute
var personsOnBobsSite = await managewell.github.io.people
  .where(':origin')
  .equals('dat://bob.com')
  .toArray()

// get the 30 oldest people indexed
var oldestPeople = await managewell.github.io.people
  .orderBy('age')
  .reverse() // oldest first
  .limit(30)
  .toArray()

// count the # of young people
var oldestPeople = await managewell.github.io.people
  .where('age')
  .belowOrEqual(18)
  .count()
```

We can also use managewell.github.io to create, modify, and delete records (and their matching files).

```js
// set the record
await managewell.github.io.people.put('dat://bob.com/person.json', {
  firstName: 'Bob',
  lastName: 'Roberts',
  age: 31
})

// update the record if it exists
await managewell.github.io.people.update('dat://bob.com/person.json', {
  age: 32
})

// update or create the record
await managewell.github.io.people.upsert('dat://bob.com/person.json', {
  age: 32
})

// delete the record
await managewell.github.io.people.delete('dat://bob.com/person.json')

// update the spelling of all Roberts records
await managewell.github.io.people
  .where('lastName')
  .equals('Roberts')
  .update({lastName: 'Robertos'})

// increment the age of all people under 18
var oldestPeople = await managewell.github.io.people
  .where('age')
  .belowOrEqual(18)
  .update(record => {
    record.age = record.age + 1
  })

// delete the 30 oldest people
var oldestPeople = await managewell.github.io.people
  .orderBy('age')
  .reverse() // oldest first
  .limit(30)
  .delete()
```

## Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [How to use managewell.github.io](#how-to-use-managewell.github.io)
  - [Table definitions](#table-definitions)
  - [Indexing sites](#indexing-sites)
  - [Creating queries](#creating-queries)
  - [Applying linear-scan filters](#applying-linear-scan-filters)
  - [Applying query modifiers](#applying-query-modifiers)
  - [Executing 'read' queries](#executing-read-queries)
  - [Executing 'write' queries](#executing-write-queries)
  - [Table helper methods](#table-helper-methods)
  - [Record methods](#record-methods)
  - [Handling multiple schemas](#handling-multiple-schemas)
  - [Preprocessing records](#preprocessing-records)
  - [Serializing records](#serializing-records)
  - [Using JSON-Schema to validate](#using-json-schema-to-validate)
  - [Helper tables](#helper-tables)
- [Class: WebDB](#class-webdb)
  - [new WebDB([name, opts])](#new-webdbname-opts)
  - [WebDB.delete([name])](#webdbdeletename)
- [Instance: managewell.github.io](#instance-managewell.github.io)
  - [webdb.open()](#webdbopen)
  - [webdb.close()](#webdbclose)
  - [webdb.delete()](#webdbdelete)
  - [webdb.define(name, definition)](#webdbdefinename-definition)
  - [webdb.indexArchive(url[, opts])](#webdbindexarchiveurl-opts)
  - [webdb.unindexArchive(url)](#webdbunindexarchiveurl)
  - [webdb.indexFile(archive, filepath)](#webdbindexfilearchive-filepath)
  - [webdb.indexFile(url)](#webdbindexfileurl)
  - [webdb.unindexFile(archive, filepath)](#webdbunindexfilearchive-filepath)
  - [webdb.unindexFile(url)](#webdbunindexfileurl)
  - [webdb.listSources()](#managewell.github.iolistsources)
  - [webdb.isSource(url)](#managewell.github.iolissourceurl)
  - [Event: 'open'](#event-open)
  - [Event: 'open-failed'](#event-open-failed)
  - [Event: 'indexes-reset'](#event-indexes-reset)
  - [Event: 'indexes-updated'](#event-indexes-updated)
  - [Event: 'source-indexing'](#event-source-indexing)
  - [Event: 'source-index-progress'](#event-source-index-progress)
  - [Event: 'source-indexed'](#event-source-indexed)
  - [Event: 'source-missing'](#event-source-missing)
  - [Event: 'source-found'](#event-source-found)
  - [Event: 'source-error'](#event-source-error)
- [Instance: managewell.github.ioTable](#instance-managewell.github.iotable)
  - [table.count()](#tablecount)
  - [table.delete(url)](#tabledeleteurl)
  - [table.each(fn)](#tableeachfn)
  - [table.filter(fn)](#tablefilterfn)
  - [table.get(url)](#tablegeturl)
  - [table.get(key, value)](#tablegetkey-value)
  - [table.isRecordFile(url)](#tableisrecordfileurl)
  - [table.limit(n)](#tablelimitn)
  - [table.listRecordFiles(url)](#tablelistrecordfilesurl)
  - [table.name](#tablename)
  - [table.offset(n)](#tableoffsetn)
  - [table.orderBy(key)](#tableorderbykey)
  - [table.put(url, record)](#tableputurl-record)
  - [table.query()](#tablequery)
  - [table.reverse()](#tablereverse)
  - [table.schema](#tableschema)
  - [table.toArray()](#tabletoarray)
  - [table.update(url, updates)](#tableupdateurl-updates)
  - [table.update(url, fn)](#tableupdateurl-fn)
  - [table.upsert(url, updates)](#tableupserturl-updates)
  - [table.upsert(url, fn)](#tableupserturl-fn)
  - [table.where(key)](#tablewherekey)
  - [Event: 'put-record'](#event-put-record)
  - [Event: 'del-record'](#event-del-record)
- [Instance: managewell.github.ioQuery](#instance-managewell.github.ioquery)
  - [query.clone()](#queryclone)
  - [query.count()](#querycount)
  - [query.delete()](#querydelete)
  - [query.each(fn)](#queryeachfn)
  - [query.eachKey(fn)](#queryeachkeyfn)
  - [query.eachUrl(fn)](#queryeachurlfn)
  - [query.filter(fn)](#queryfilterfn)
  - [query.first()](#queryfirst)
  - [query.keys()](#querykeys)
  - [query.last()](#querylast)
  - [query.limit(n)](#querylimitn)
  - [query.offset(n)](#queryoffsetn)
  - [query.orderBy(key)](#queryorderbykey)
  - [query.put(record)](#queryputrecord)
  - [query.urls()](#queryurls)
  - [query.reverse()](#queryreverse)
  - [query.toArray()](#querytoarray)
  - [query.uniqueKeys()](#queryuniquekeys)
  - [query.until(fn)](#queryuntilfn)
  - [query.update(updates)](#queryupdateupdates)
  - [query.update(fn)](#queryupdatefn)
  - [query.where(key)](#querywherekey)
- [Instance: managewell.github.ioWhereClause](#instance-managewell.github.iowhereclause)
  - [where.above(value)](#whereabovevalue)
  - [where.aboveOrEqual(value)](#whereaboveorequalvalue)
  - [where.anyOf(values)](#whereanyofvalues)
  - [where.anyOfIgnoreCase(values)](#whereanyofignorecasevalues)
  - [where.below(value)](#wherebelowvalue)
  - [where.belowOrEqual(value)](#wherebeloworequalvalue)
  - [where.between(lowerValue, upperValue[, options])](#wherebetweenlowervalue-uppervalue-options)
  - [where.equals(value)](#whereequalsvalue)
  - [where.equalsIgnoreCase(value)](#whereequalsignorecasevalue)
  - [where.noneOf(values)](#wherenoneofvalues)
  - [where.notEqual(value)](#wherenotequalvalue)
  - [where.startsWith(value)](#wherestartswithvalue)
  - [where.startsWithAnyOf(values)](#wherestartswithanyofvalues)
  - [where.startsWithAnyOfIgnoreCase(values)](#wherestartswithanyofignorecasevalues)
  - [where.startsWithIgnoreCase(value)](#wherestartswithignorecasevalue)
- [How it works](#how-it-works)
  - [Why not put all records in one file?](#why-not-put-all-records-in-one-file)
    - [Performance](#performance)
    - [Linkability](#linkability)
- [Changelog](#changelog)
  - [4.1.0](#410)
  - [4.0.0](#400)
  - [3.0.0](#300)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## How to use managewell.github.io

### Table definitions

Use the [`define()`](#managewell.github.iodefinename-definition) method to define your tables, and then call [`managewell.github.io.open()`](#managewell.github.ioopen) to create them.

### Indexing sites

Use [`indexArchive()`](#managewell.github.ioindexarchiveurl-opts) and [`unindexArchive()`](#managewell.github.iounindexarchiveurl) to control which sites will be indexed.
Indexed data will persist in the database until `unindexArchive()` is called.
However, `indexArchive()` should always be called on load to get the latest data.

If you only want to index the current state of a site, and do not want to watch for updates, call `indexArchive()` with the `{watch: false}` option.

You can index and de-index individual files using [`indexFile()`](#managewell.github.ioindexfileurl) and [`unindexFile()`](#managewell.github.iounindexfileurl).

### Creating queries

Queries are created with a chained function API.
You can create a query from the table object using [`.query()`](#tablequery), [`.where()`](#tablewherekey), or [`.orderBy()`](#tableorderbykey).
The `where()` method returns an object with [multiple filter functions that you can use](#instance-managewell.github.iowhereclause).

```js
var myQuery = managewell.github.io.query().where('foo').equals('bar')
var myQuery = managewell.github.io.where('foo').equals('bar') // equivalent
var myQuery = managewell.github.io.where('foo').startsWith('ba')
var myQuery = managewell.github.io.where('foo').between('bar', 'baz', {includeLower: true, includeUpper: false})
```

Each query has a primary key.
By default, this is the `url` attribute, but it can be changed using [`.where()`](#querywherekey) or [`.orderBy()`](#queryorderbykey).
In this example, the primary key becomes 'foo':

```js
var myQuery = managewell.github.io.orderBy('foo')
```

At this time, the primary key must be one of the indexed attributes.
There are 2 indexes created automatically for every record: `url` and `origin`.
The other indexes are specified in your table's [`define()`](#managewell.github.iodefinename-definition) call using the `index` option.

### Applying linear-scan filters

After the primary key index is applied, you can apply additional filters using [filter(fn)](#queryfilterfn) and [until(fn)](#queryuntilfn).
These methods are called "linear scan" filters because they require each record to be loaded and then filtered out.
(Until stops when it hits the first `false` response.)

```js
var myQuery = managewell.github.io.query()
  .where('foo').equals('bar')
  .filter(record => record.beep == 'boop') // additional filter
```

### Applying query modifiers

You can apply the following modifiers to your query to alter the output:

  - [limit(n)](#querylimitn)
  - [offset(n)](#queryoffsetn)
  - [reverse()](#queryreverse)

### Executing 'read' queries

Once your query has been defined, you can execute and read the results using one of these methods:

  - [count()](#querycount)
  - [each(fn)](#queryeachfn)
  - [eachKey(fn)](#queryeachkeyfn)
  - [eachUrl(fn)](#queryeachurlfn)
  - [first()](#queryfirst)
  - [keys()](#querykeys)
  - [last()](#querylast)
  - [urls()](#queryurls)
  - [toArray()](#querytoarray)
  - [uniqueKeys()](#queryuniquekeys)

### Executing 'write' queries

Once your query has been defined, you can execute and *modify* the results using one of these methods:

  - [delete()](#querydelete)
  - [put(record)](#queryputrecord)
  - [update(updates)](#queryupdateupdates)
  - [update(fn)](#queryupdatefn)
  
If you try to modify rows in archives that are not writable, WebDB will throw an error.

### Table helper methods

The following methods exist on the table object for query reads and writes:

  - [table.delete(url)](#tabledeleteurl)
  - [table.each(fn)](#tableeachfn)
  - [table.get(url)](#tablegeturl)
  - [table.get(key, value)](#tablegetkey-value)
  - [table.put(url, record)](#tableputurl-record)
  - [table.toArray()](#tabletoarray)
  - [table.update(url, updates)](#tableupdateurl-updates)
  - [table.update(url, fn)](#tableupdateurl-fn)
  - [table.upsert(url, updates)](#tableupserturl-updates)
  - [table.upsert(url, fn)](#tableupserturl-fn)

### Record methods

```js
record.getRecordURL()    // => 'dat://foo.com/bar.json'
record.getRecordOrigin() // => 'dat://foo.com'
record.getIndexedAt()    // => 1511913554723
```

Every record is emitted in a wrapper object with the following methods:

 - `getRecordURL()` The URL of the record.
 - `getRecordOrigin()` The URL of the site the record was found on.
 - `getIndexedAt()` The timestamp of when the record was indexed.

These attributes can be used in indexes with the following IDs:

 - `:url`
 - `:origin`
 - `:indexedAt`

For instance:

```js
webdb.define('things', {
  // ...
  index: [
    ':indexedAt', // ordered by time the record was indexed
    ':origin+createdAt' // ordered by origin and declared create timestamp (a record attribute)
  ],
  // ...
})
await managewell.github.io.open()
webdb.things.where(':indexedAt').above(Date.now() - ms('1 week'))
webdb.things.where(':origin+createdAt').between(['dat://bob.com', 0], ['dat://bob.com', Infinity])
```

### Handling multiple schemas

Since the Web is a complex place, you'll frequently have to deal with multiple schemas which are slightly different.
To deal with this, you can use a definition object to support multiple attribute names under one index.

```js
managewell.github.io.define('places', {
  // ...

  index: [
    // a simple index definition:
    'name',

    // an object index definition:
    {name: 'zipCode', def: ['zipCode', 'zip_code']}
  ]

  // ...
})
```

Now, when you run queries on the `'zipCode'` key, you will search against both `'zipCode'` and `'zip_code'`.
Note, however, that the records emitted from the query will not be changed by managewell.github.io and so they may differ.

For example:

```js
webdb.places.where('zipCode').equals('78705').each(record => {
  console.log(record.zipCode) // may be '78705' or undefined
  console.log(record.zip_code) // may be '78705' or undefined
})
```

To solve this, you can [preprocess records](#preprocessing-records).

### Preprocessing records

Sometimes, you need to modify records before they're stored in the database. This can be for a number of reasons:

 - Normalization. Small differences in accepted record schemas may need to be merged (see [handling multiple schemas](#handling-multiple-schemas)).
 - Indexing. managewell.github.io's index spec only supports toplevel attributes. If the data is embedded in a sub-object, you'll need to place the data at the top-level.
 - Computed attributes.

For these cases, you can use the `preprocess(record)` function in the table definition:

```js
managewell.github.io.define('places', {
  // ...

  preprocess(record) {
    // normalize zipCode and zip_code
    if (record.zip_code) {
      record.zipCode = record.zip_code
    }

    // move an attribute to the root object for indexing
    record.title = record.info.title

    // compute an attribute
    record.location = `${record.address} ${record.city}, ${record.state} ${record.zipCode}`

    return record
  }

  // ...
})
```

These attributes will be stored in the managewell.github.io table.

### Serializing records

When records are updated by managewell.github.io, they are published to a Dat site as a file.
Since these files are distributed on the Web, it's wise to avoid adding noise to the record.

To control the exact record that will be published, you can set the `serialize(record)` function in the table definition:

```js
managewell.github.io.define('places', {
  // ...

  serialize(record) {
    // write the following object to the dat site:
    return {
      info: record.info,
      city: record.city,
      state: record.state,
      zipCode: record.zipCode
    }
  }

  // ...
})
```

### Using JSON-Schema to validate

The default way to validate records is to provide a validator function.
If the function throws an error or returns falsy, the record will not be indexed.

It can be tedious to write validation functions, so you might want to use JSON-Schema:

```js
const Ajv = require('ajv')
webdb.define('people', {
  validate: (new Ajv()).compile({
    type: 'object',
    properties: {
      firstName: {
        type: 'string'
      },
      lastName: {
        type: 'string'
      },
      age: {
        description: 'Age in years',
        type: 'integer',
        minimum: 0
      }
    },
    required: ['firstName', 'lastName']
  }),

  // ...
})
```

### Helper tables

Sometimes you need internal storage to help you maintain application state.
This may be for interfaces, or data which is private, or for special kinds of indexes.

For instance, in the [Managewell](https://github.com/managewell/managewell.github.io) app, we needed an index for notifications.
This index was conditional: it needed to contain posts which were replies to the user, or likes which were on the user's post.
For cases like this, you can use a "helper table."

```js
webdb.define('notifications', {
  helperTable: true,
  index: ['createdAt'],
  preprocess (record) {
    record.createdAt = Date.now()
  }
})
```

When the `helperTable` attribute is set to `true` in a table definition, the table will not be used to index Dat archives.
Instead, it will exist purely in the local data cache, and only contain data which is `.put()` there.
In all other respects, it behaves like a normal table.

In [Fritter](https://github.com/beakerbrowser/fritter), the helper table is used with events to track notifications:

```js
// track reply notifications
webdb.posts.on('put', async ({record, url, origin}) => {
  if (origin === userUrl) return // dont index the user's own posts
  if (isAReplyToUser(record) === false) return // only index replies to the user
  if (await isNotificationIndexed(url)) return // don't index if already indexed
  await db.notifications.put(url, {type: 'reply', url})
})
webdb.posts.on('del', async ({url}) => {
  if (await isNotificationIndexed(url)) {
    await db.notifications.delete(url)
  }
})
```

## Class: managewell.github.io

### new managewell.github.io([name, opts])

```js
var webdb = new WebDB('mydb')
```

 - `name` String. Defaults to `'managewell.github.io'`. If run in the browser, this will be the name of the IndexedDB instance. If run in NodeJS, this will be the path of the LevelDB folder.
 - `opts` Object.
   - `DatArchive` Constructor. The class constructor for dat archive instances. If in node, you should specify [node-dat-archive](https://npm.im/node-dat-archive).

Create a new `managewell.github.io` instance.
The given `name` will control where the indexes are saved.
You can specify different names to run multiple WebDB instances at once.

### managewell.github.io.delete([name])

```js
await managewell.github.io.delete('mydb')
```

 - `name` String. Defaults to `'managewell.github.io'`. If run in the browser, this will be the name of the IndexedDB instance. If run in NodeJS, this will be the path of the LevelDB folder.
 - Returns Promise&lt;Void&gt;.

Deletes the indexes and metadata for the given WebDB.

## Instance: managewell.github.io

### managewell.github.io.open()

```js
await webdb.open()
```

 - Returns Promise&lt;Object&gt;.
   - `rebuilds` Array&lt;String&gt;. The tables which were built or rebuilt during setup.

Runs final setup for the WebDB instance.
This must be run after [`.define()`](#managewell.github.iodefinename-definition) to create the table instances.

### managewell.github.io.close()

```js
await managewell.github.io.close()
```

 - Returns Promise&lt;Void&gt;.

Closes the managewell.github.io instance.

### managewell.github.io.delete()

```js
await managewell.github.io.delete()
```

 - Returns Promise&lt;Void&gt;.

Closes and destroys all indexes in the managewell.github.io instance.

You can `.delete()` and then `.open()` a WebDB to recreate its indexes.

```js
await managewell.github.io.delete()
await managewell.github.io.open()
```

### managewell.github.io.define(name, definition)

 - `name` String. The name of the table.
 - `definition` Object.
   - `index` Array&lt;String or Object&gt;. A list of attributes which should have secondary indexes produced for querying. Each `index` value is a keypath (see https://www.w3.org/TR/IndexedDB/#dfn-key-path) or an object definition (see below).
   - `filePattern` String or Array&lt;String&gt;. An [anymatch](https://www.npmjs.com/package/anymatch) list of files to index.
   - `helperTable` Boolean. If true, the table will be used for storing internal data and will not be used to index Dat archives. See [helper tables](#helper-tables)
   - `validate` Function. A method to accept or reject a file from indexing based on its content. If the method returns falsy or throws an error, the file will not be indexed.
     - `record` Object.
     - Returns Boolean.
   - `preprocess` Function. A method to modify the record after read from the dat site. See [preprocessing records](#preprocessing-records).
     - `record` Object.
     - Returns Object.
   - `serialize` Function. A method to modify the record before write to the dat site. See [serializing records](#serializing-records).
     - `record` Object.
     - Returns Object.
 - Returns Void.

Creates a new table on the `managewell.github.io` object.
The table will be set at `webdb.{name}` and be the `managewell.github.ioTable` type.
This method must be called before [`open()`](#webdbopen)

Indexed attributes may either be defined as a [keypath string](https://www.w3.org/TR/IndexedDB/#dfn-key-path) or an object definition.
The object definition has the following values:

 - `name` String. The name of the index.
 - `def` String or Array&lt;String&gt;. The definition of the index.

If the value of `def` is an array, it supports each definition.
This is useful when supporting multiple schemas ([learn more here](#handling-multiple-schemas)).

In the index definition, you can specify compound indexes with a `+` separator in the keypath.
You can also index each value of an array using the `*` sigil at the start of the name.
Some example index definitions:

```
a simple index           - 'firstName'
as an object def         - {name: 'firstName', def: 'firstName'}
a compound index         - 'firstName+lastName'
index an array's values  - '*favoriteFruits'
many keys                - {name: 'firstName', def: ['firstName', 'first_name']}
many keys, compound      - {name: 'firstName+lastName', def: ['firstName+lastName', 'first_name+last_name']}
```

You can specify which files should be processed into the table using the `filePattern` option.
If unspecified, it will default to all json files on the site (`'*.json'`).

Example:

```js
webdb.define('people', {
  validate(record) {
    assert(record.firstName && typeof record.firstName === 'string')
    assert(record.lastName && typeof record.lastName === 'string')
    return true
  },
  index: ['lastName', 'lastName+firstName', 'age'],
  filePattern: [
    '/person.json',
    '/people/*.json'
  ]
})

await managewell.github.io.open()
// the new table will now be defined at managewell.github.io.people
```

### managewell.github.io.indexArchive(url[, opts])

```js
await managewell.github.io.indexArchive('dat://foo.com')
```

 - `url` String or DatArchive or Array&lt;String or DatArchive&gt;. The sites to index.
 - `opts` Object.
   - `watch` Boolean. Should managewell.github.io watch the archive for changes, and index them immediately? Defaults to true.
 - Returns Promise&lt;Void&gt;.

Add one or more dat:// sites to be indexed.
The method will return when the site has been fully indexed.
This will add the given archive to the "sources" list.

### managewell.github.io.unindexArchive(url)

```js
await managewell.github.io.unindexArchive('dat://foo.com')
```

 - `url` String or DatArchive. The site to deindex.
 - Returns Promise&lt;Void&gt;.

Remove a dat:// site from the dataset.
The method will return when the site has been fully de-indexed.
This will remove the given archive from the "sources" list.

### webdb.indexFile(archive, filepath)

```js
await managewell.github.io.indexFile(fooArchive, '/bar.json')
```

 - `archive` DatArchive. The site containing the file to index.
 - `filepath` String. The path of the file to index.
 - Returns Promise&lt;Void&gt;.

Add a single file to the index.
The method will return when the file has been indexed.

This will not add the file or its archive to the "sources" list.
Unlike `indexArchive`, managewell.github.io will not watch the file after this call.

### managewell.github.io.indexFile(url)

```js
await managewell.github.io.indexFile('dat://foo.com/bar.json')
```

 - `url` String. The url of the file to index.
 - Returns Promise&lt;Void&gt;.

Add a single file to the index.
The method will return when the file has been indexed.

This will not add the file or its archive to the "sources" list.

### managewell.github.io.unindexFile(archive, filepath)

```js
await managewell.github.io.unindexFile(fooArchive, '/bar.json')
```

 - `archive` DatArchive. The site containing the file to deindex.
 - `filepath` String. The path of the file to deindex.
 - Returns Promise&lt;Void&gt;.

Remove a single file from the dataset.
The method will return when the file has been de-indexed.

### managewell.github.io.unindexFile(url)

```js
await managewell.github.io.unindexFile('dat://foo.com')
```

 - `url` String. The url of the file to deindex.
 - Returns Promise&lt;Void&gt;.

Remove a single file from the dataset.
The method will return when the file has been de-indexed.

### managewell.github.io.listSources()

```js
var urls = await managewell.github.io.listSources()
```

 - Returns Array&lt;String&gt;.

Lists the URLs of the dat:// sites which are included in the dataset.

### managewell.github.io.isSource(url)

```js
var urls = await managewell.github.io.isSource('dat://foo.com')
```

 - Returns Boolean.

Is the given dat:// URL included in the dataset?

### Event: 'open'

```js
webdb.on('open', () => {
  console.log('managewell.github.io is ready for use')
})
```

Emitted when the managewell.github.io instance has been opened using [`open()`](#webdbopen).

### Event: 'open-failed'

```js
managewell.github.io.on('open-failed', (err) => {
  console.log('managewell.github.io failed to open', err)
})
```

 - `error` Error.

Emitted when the managewell.github.io instance fails to open during [`open()`](#webdbopen).

### Event: 'indexes-reset'

```js
managewell.github.io.on('indexes-reset', () => {
  console.log('managewell.github.io detected a change in schemas and reset all indexes')
})
```

Emitted when the managewell.github.io instance detects a change in the schemas and has to reindex the dataset.
All indexes are cleared and will be reindexed as sources are added.

### Event: 'indexes-updated'

```js
managewell.github.io.on('indexes-updated', (url, version) => {
  console.log('Tables were updated for', url, 'at version', version)
})
```

 - `url` String. The archive that was updated.
 - `version` Number. The version which was updated to.

Emitted when the managewell.github.io instance has updated the stored data for a archive.

### Event: 'source-indexing'

```js
managewell.github.io.on('source-indexing', (url, startVersion, targetVersion) => {
  console.log('Tables are updating for', url, 'from version', startVersion, 'to', targetVersion)
})
```

 - `url` String. The archive that was updated.
 - `startVersion` Number. The version which is being indexed from.
 - `targetVersion` Number. The version which is being indexed to.

Emitted when the managewell.github.io instance has started to index the given archive.

### Event: 'source-index-progress'

```js
managewell.github.io.on('source-index-progress', (url, tick, total) => {
  console.log('Update for', url, 'is', Math.round(tick / total * 100), '% complete')
})
```

 - `url` String. The archive that was updated.
 - `tick` Number. The current update being applied.
 - `total` Number. The total number of updates being applied.

Emitted when an update has been applied during an indexing process.

### Event: 'source-indexed'

```js
managewell.github.io.on('source-indexed', (url, version) => {
  console.log('Tables were updated for', url, 'at version', version)
})
```

 - `url` String. The archive that was updated.
 - `version` Number. The version which was updated to.

Emitted when the managewell.github.io instance has indexed the given archive.
This is similar to `'indexes-updated'`, but it fires every time a source is indexed, whether or not it results in updates to the indexes.

### Event: 'source-missing'

```js
managewell.github.io.on('source-missing', (url) => {
  console.log('managewell.github.io couldnt find', url, '- now searching')
})
```

Emitted when a source's data was not locally available or found on the network.
When this occurs, managewell.github.io will continue searching for the data, and emit `'source-found'` on success.

### Event: 'source-found'

```js
managewell.github.io.on('source-found', (url) => {
  console.log('managewell.github.io has found and indexed', url)
})
```

Emitted when a source's data was found after originally not being found during indexing.
This event will only be emitted after `'source-missing'` is emitted.

### Event: 'source-error'

```js
managewell.github.io.on('source-error', (url, err) => {
  console.log('managewell.github.io failed to index', url, err)
})
```

Emitted when a source fails to load.

## Instance: managewell.github.ioTable

### table.count()

```js
var numRecords = await managewell.github.io.mytable.count()
```

 - Returns Promise&lt;Number&gt;.

Count the number of records in the table.

### table.delete(url)

```js
await managewell.github.io.mytable.delete('dat://foo.com/bar.json')
```

 - Returns Promise&lt;Number&gt;. The number of deleted records (should be 0 or 1).

Delete the record at the given URL.

### table.each(fn)

```js
await managewell.github.io.mytable.each(record => {
  console.log(record)
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Void.
 - Returns Promise&lt;Void&gt;.

Iterate over all records in the table with the given function.

### table.filter(fn)

```js
var records = await managewell.github.io.mytable.filter(record => {
  return (record.foo == 'bar')
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Boolean.
 - Returns managewell.github.ioQuery.

Start a new query and apply the given filter function to the resultset.

### table.get(url)

```js
var record = await managewell.github.io.mytable.get('dat://foo.com/myrecord.json')
```

 - `url` String. The URL of the record to fetch.
 - Returns Promise&lt;Object&gt;.

Get the record at the given URL.

### table.get(key, value)

```js
var record = await webdb.mytable.get('foo', 'bar')
```

 - `key` String. The keyname to search against.
 - `value` Any. The value to match against.
 - Promise&lt;Object&gt;.
 
Get the record first record to match the given key/value query.

### table.isRecordFile(url)

```js
var isRecord = webdb.mytable.isRecordFile('dat://foo.com/myrecord.json')
```

 - `url` String.
 - Returns Boolean.

Tells you whether the given URL matches the table's file pattern.

### table.limit(n)

```js
var query = managewell.github.io.mytable.limit(10)
```

 - `n` Number.
 - Returns managewell.github.ioQuery.

Creates a new query with the given limit applied.

### table.listRecordFiles(url)

```js
var recordFiles = await managewell.github.io.mytable.listRecordFiles('dat://foo.com')
```

 - `url` String.
 - Returns Promise&lt;Array&lt;Object&gt;&gt;. On each object:
   - `recordUrl` String.
   - `table` managewell.github.ioTable.

Lists all files on the given URL which match the table's file pattern.

### table.name

 - String.

The name of the table.

### table.offset(n)

```js
var query = managewell.github.io.mytable.offset(5)
```

 - `n` Number.
 - Returns managewell.github.ioQuery.

Creates a new query with the given offset applied.

### table.orderBy(key)

```js
var query = managewell.github.io.mytable.orderBy('foo')
```

 - `key` String.
 - Returns WebDBQuery.

Creates a new query ordered by the given key.

### table.put(url, record)

```js
await managewell.github.io.mytable.put('dat://foo.com/myrecord.json', {foo: 'bar'})
```

 - `url` String.
 - `record` Object.
 - Returns Promise&lt;String&gt;. The URL of the written record.

Replaces or creates the record at the given URL with the `record`.

### table.query()

```js
var query = managewell.github.io.mytable.query()
```

 - Returns managewell.github.ioQuery.

Creates a new query.

### table.reverse()

```js
var query = managewell.github.io.mytable.reverse()
```

 - Returns managewell.github.ioQuery.

Creates a new query with reverse-order applied.

### table.schema

 - Object.

The schema definition for the table.

### table.toArray()

```js
var records = await managewell.github.io.mytable.toArray()
```

 - Returns Promise&lt;Array&lt;Object&gt;&gt;.

Returns an array of all records in the table.

### table.update(url, updates)

```js
var wasUpdated = await managewell.github.io.mytable.update('dat://foo.com/myrecord.json', {foo: 'bar'})
```

 - `url` String. The record to update.
 - `updates` Object. The new values to set on the record.
 - Returns Promise&lt;Number&gt;. The number of records updated.

Updates the target record with the given key values, if it exists.

### table.update(url, fn)

```js
var wasUpdated = await managewell.github.io.mytable.update('dat://foo.com/myrecord.json', record => {
  record.foo = 'bar'
  return record
})
```

 - `url` String. The record to update.
 - `fn` Function. A method to modify the record.
   - `record` Object. The record to modify.
   - Returns Object.
 - Returns Promise&lt;Number&gt;. The number of records updated.

Updates the target record with the given function, if it exists.

### table.upsert(url, updates)

```js
var didCreateNew = await managewell.github.io.mytable.upsert('dat://foo.com/myrecord.json', {foo: 'bar'})
```

 - `url` String. The record to update.
 - `updates` Object. The new values to set on the record.
 - Returns Promise&lt;Number&gt;. The number of records updated.

If a record exists at the target URL, will update it with the given key values.
If a record does not exist, will create the record.

### table.upsert(url, fn)

```js
var didCreateNew = await managewell.github.io.mytable.upsert('dat://foo.com/myrecord.json', record => {
  if (record) {
    // update
    record.foo = 'bar'
    return record
  }
  // create
  return {foo: 'bar'}
})
```

 - `url` String. The record to update.
 - `fn` Function. A method to modify the record.
   - `record` Object. The record to modify. Will be falsy if the record does ot previously exist
   - Returns Object.
 - Returns Promise&lt;Number&gt;. The number of records updated.

Updates the target record with the given function, if it exists.
If a record does not exist, will give a falsy value to the method.

### table.where(key)

```js
var whereClause = managewell.github.io.mytable.where('foo')
```

 - `key` String.
 - Returns WebDBWhereClause.

Creates a new where-clause using the given key.

### Event: 'put-record'

```js
webdb.mytable.on('put-record', ({url, origin, indexedAt, record}) => {
  console.log('Table was updated for', url, '(origin:', origin, ') at ', indexedAt)
  console.log('Record data:', record)
})
```

 - `url` String. The url of the record that was updated.
 - `origin` String. The url origin of the record that was updated.
 - `indexedAt` Number. The timestamp of the index update.
 - `record` Object. The content of the updated record.

Emitted when the table has updated the stored data for a record.

### Event: 'del-record'

```js
webdb.mytable.on('del-record', ({url, origin, indexedAt}) => {
  console.log('Table was updated for', url, '(origin:', origin, ') at ', indexedAt)
})
```

 - `url` String. The url of the record that was deleted.
 - `origin` String. The url origin of the record that was deleted.
 - `indexedAt` Number. The timestamp of the index update.

Emitted when the table has deleted the stored data for a record.
This can happen because the record has been deleted, or because a new version of the record fails validation.

## Instance: managewell.github.ioQuery

### query.clone()

```js
var query = managewell.github.io.mytable.query().clone()
```

 - Returns managewell.github.ioQuery.

Creates a copy of the query.

### query.count()

```js
var numRecords = await managewell.github.io.mytable.query().count()
```

 - Returns Promise&lt;Number&gt;. The number of found records.

Gives the count of records which match the query.

### query.delete()

```js
var numDeleted = await managewell.github.io.mytable.query().delete()
```

 - Returns Promise&lt;Number&gt;. The number of deleted records.

Deletes all records which match the query.

### query.each(fn)

```js
await managewell.github.io.mytable.query().each(record => {
  console.log(record)
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Void.
 - Returns Promise&lt;Void&gt;.

Calls the given function with all records which match the query.

### query.eachKey(fn)

```js
await managewell.github.io.mytable.query().eachKey(url => {
  console.log('URL =', url)
})
```

 - `fn` Function.
   - `key` String.
   - Returns Void.
 - Returns Promise&lt;Void&gt;.

Calls the given function with the value of the query's primary key for each matching record.

The `key` is determined by the index being used.
By default, this is the `url` attribute, but it can be changed by using `where()` or `orderBy()`.

Example:

```js
await managewell.github.io.mytable.orderBy('age').eachKey(age => {
  console.log('Age =', age)
})
```

### query.eachUrl(fn)

```js
await managewell.github.io.mytable.query().eachUrl(url => {
  console.log('URL =', url)
})
```

 - `fn` Function.
   - `url` String.
   - Returns Void.
 - Returns Promise&lt;Void&gt;.

Calls the given function with the URL of each matching record.

### query.filter(fn)

```js
var query = managewell.github.io.mytable.query().filter(record => {
  return record.foo == 'bar'
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Boolean.
 - Returns WebDBQuery.

Applies an additional filter on the query.

### query.first()

```js
var record = await managewell.github.io.mytable.query().first()
```

 - Returns Promise&lt;Object&gt;.

Returns the first result in the query.

### query.keys()

```js
var keys = await managewell.github.io.mytable.query().keys()
```

 - Returns Promise&lt;Array&lt;String&gt;&gt;.

Returns the value of the primary key for each matching record.

The `key` is determined by the index being used.
By default, this is the `url` attribute, but it can be changed by using `where()` or `orderBy()`.

```js
var ages = await managewell.github.io.mytable.orderBy('age').keys()
```

### query.last()

```js
var record = await managewell.github.io.mytable.query().last()
```

 - Returns Promise&lt;Object&gt;.

Returns the last result in the query.

### query.limit(n)

```js
var query = managewell.github.io.mytable.query().limit(10)
```

 - `n` Number.
 - Returns managewell.github.ioQuery.

Limits the number of matching record to the given number.

### query.offset(n)

```js
var query = managewell.github.io.mytable.query().offset(10)
```

 - `n` Number.
 - Returns managewell.github.ioQuery.

Skips the given number of matching records.

### query.orderBy(key)

```js
var query = managewell.github.io.mytable.query().orderBy('foo')
```

 - `key` String.
 - Returns managewell.github.ioQuery.

Sets the primary key and sets the resulting order to match its values.

### query.put(record)

```js
var numWritten = await managewell.github.io.mytable.query().put({foo: 'bar'})
```

 - `record` Object.
 - Returns Promise&lt;Number&gt;. The number of written records.

Replaces each matching record with the given value.

### query.urls()

```js
var urls = await managewell.github.io.mytable.query().urls()
```

 - Returns Promise&lt;Array&lt;String&gt;&gt;.

Returns the url of each matching record.

### query.reverse()

```js
var query = managewell.github.io.mytable.query().reverse()
```

 - Returns managewell.github.ioQuery.

Reverses the order of the results.

### query.toArray()

```js
var records = await managewell.github.io.mytable.query().toArray()
```

 - Returns Promise&lt;Array&lt;Object&gt;&gt;.

Returns the value of each matching record.

### query.uniqueKeys()

```js
var keys = await managewell.github.io.mytable.query().uniqueKeys()
```

 - Returns Promise&lt;Array&lt;String&gt;&gt;.

Returns the value of the primary key for each matching record, with duplicates filtered out.

The `key` is determined by the index being used.
By default, this is the `url` attribute, but it can be changed by using `where()` or `orderBy()`.

Example: 

```js
var ages = await managewell.github.io.mytable.orderBy('age').uniqueKeys()
```

### query.until(fn)

```js
var query = managewell.github.io.mytable.query().until(record => {
  return record.foo == 'bar'
})
```

 - `fn` Function.
   - `record` Object.
   - Returns Boolean.
 - Returns managewell.github.ioQuery.

Stops emitting matching records when the given function returns true.

### query.update(updates)

```js
var numUpdated = await managewell.github.io.mytable.query().update({foo: 'bar'})
```

 - `updates` Object. The new values to set on the record.
 - Returns Promise&lt;Number&gt;. The number of updated records.

Updates all matching record with the given values.

### query.update(fn)

```js
var numUpdated = await managewell.github.io.mytable.query().update(record => {
  record.foo = 'bar'
  return record
})
```

 - `fn` Function. A method to modify the record.
   - `record` Object. The record to modify.
   - Returns Object.
 - Returns Promise&lt;Number&gt;. The number of updated records.

Updates all matching record with the given function.

### query.where(key)

```js
var whereClause = managewell.github.io.mytable.query().where('foo')
```

 - `key` String. The attribute to query against.
 - Returns managewell.github.ioWhereClause.

Creates a new where clause.

## Instance: managewell.github.ioWhereClause

### where.above(value)

```js
var query = managewell.github.io.mytable.query().where('foo').above('bar')
var query = managewell.github.io.mytable.query().where('age').above(18)
```

 - `value` Any. The lower bound of the query.
 - Returns managewell.github.ioQuery.

### where.aboveOrEqual(value)

```js
var query = managewell.github.io.mytable.query().where('foo').aboveOrEqual('bar')
var query = managewell.github.io.mytable.query().where('age').aboveOrEqual(18)
```

 - `value` Any. The lower bound of the query.
 - Returns managewell.github.ioQuery.

### where.anyOf(values)

```js
var query = managewell.github.io.mytable.query().where('foo').anyOf(['bar', 'baz'])
```

 - `values` Array&lt;Any&gt;.
 - Returns managewell.github.ioQuery.

Does not work on compound indexes.

### where.anyOfIgnoreCase(values)

```js
var query = managewell.github.io.mytable.query().where('foo').anyOfIgnoreCase(['bar', 'baz'])
```

 - `values` Array&lt;Any&gt;.
 - Returns managewell.github.ioQuery.

Does not work on compound indexes.

### where.below(value)

```js
var query = managewell.github.io.mytable.query().where('foo').below('bar')
var query = managewell.github.io.mytable.query().where('age').below(18)
```

 - `value` Any. The upper bound of the query.
 - Returns managewell.github.ioQuery.

### where.belowOrEqual(value)

```js
var query = managewell.github.io.mytable.query().where('foo').belowOrEqual('bar')
var query = managewell.github.io.mytable.query().where('age').belowOrEqual(18)
```

 - `value` Any. The upper bound of the query.
 - Returns managewell.github.ioQuery.

### where.between(lowerValue, upperValue[, options])

```js
var query = managewell.github.io.mytable.query().where('foo').between('bar', 'baz', {includeUpper: true, includeLower: true})
var query = managewell.github.io.mytable.query().where('age').between(18, 55, {includeLower: true})
```

 - `lowerValue` Any.
 - `upperValue` Any.
 - `options` Object.
   - `includeUpper` Boolean.
   - `includeLower` Boolean.
 - Returns managewell.github.ioQuery.

### where.equals(value)

```js
var query = managewell.github.io.mytable.query().where('foo').equals('bar')
```

 - `value` Any.
 - Returns managewell.github.ioQuery.

### where.equalsIgnoreCase(value)

```js
var query = managewell.github.io.mytable.query().where('foo').equalsIgnoreCase('bar')
```

 - `value` Any.
 - Returns managewell.github.ioQuery.

Does not work on compound indexes.

### where.noneOf(values)

```js
var query = managewell.github.io.mytable.query().where('foo').noneOf(['bar', 'baz'])
```

 - `values` Array&lt;Any&gt;.
 - Returns managewell.github.ioQuery.

Does not work on compound indexes.

### where.notEqual(value)

```js
var query = managewell.github.io.mytable.query().where('foo').notEqual('bar')
```

 - `value` Any.
 - Returns managewell.github.ioQuery.

Does not work on compound indexes.

### where.startsWith(value)

```js
var query = managewell.github.io.mytable.query().where('foo').startsWith('ba')
```

 - `value` Any.
 - Returns managewell.github.ioQuery.

Does not work on compound indexes.

### where.startsWithAnyOf(values)

```js
var query = managewell.github.io.mytable.query().where('foo').startsWithAnyOf(['ba', 'bu'])
```

 - `values` Array&lt;Any&gt;.
 - Returns managewell.github.ioQuery.

Does not work on compound indexes.

### where.startsWithAnyOfIgnoreCase(values)

```js
var query = managewell.github.io.mytable.query().where('foo').startsWithAnyOfIgnoreCase(['ba', 'bu'])
```

 - `values` Array&lt;Any&gt;.
 - Returns managewell.github.ioQuery.

Does not work on compound indexes.

### where.startsWithIgnoreCase(value)

```js
var query = managewell.github.io.mytable.query().where('foo').startsWithIgnoreCase('ba')
```

 - `value` Any.
 - Returns managewell.github.ioQuery.

Does not work on compound indexes.


## How does it work?

managewell.github.io abstracts over the [DatArchive API](https://beakerbrowser.com/docs/apis/dat.html) to provide a simple database-like interface. It's inspired by [Dexie.js](https://github.com/dfahlander/Dexie.js) and built using [LevelDB](https://github.com/Level/level). (In the browser, it runs on IndexedDB using [level.js](https://github.com/maxogden/level.js).

managewell.github.io scans a set of source Dat archives for files that match a path pattern. Web DB caches and indexes those files so they can be queried easily and quickly. managewell.github.io also provides a simple interface for adding, editing, and removing records from archives.

managewell.github.io sits on top of Dat archives. It duplicates ingested data into IndexedDB, which acts as a throwaway cache. The cached data can be reconstructed at any time from the source Dat archives.

managewell.github.io treats individual files in the Dat archive as individual records in a table. As a result, there's a direct mapping for each table to a folder of JSON files. For instance, if you had a `posts` table, it might map to the `/posts/*.json` files. managewell.github.io's mutators, e.g., `put`, `add`, `update`, simply writes records as JSON files in the `posts/` directory. managewell.github.io's readers and query-ers, like `get()` and `where()`, read from the IndexedDB cache.

managewell.github.io watches its source archives for changes to the JSON files that compose its records. When the files change, it syncs and reads the changes, then updates IndexedDB, keeping query results up-to-date. Roughly, the flow is: `put() -> archive/posts/12345.json -> indexer -> indexeddb -> get()`.

### Why not put all records in one file?

Storing records in one file—`posts.json` for example—is an intuitive way to manage data on the peer-to-peer Web, but putting each record in an individual file is a much better choice for performance and linkability.

#### Performance

The `dat://` protocol doesn't support partial updates at the file-level, which means that with multiple records in a single file, every time a user adds a record, anyone who follows that user must sync and re-download the *entire* file. As the file continues to grow, performance will degrade. Putting each record in an individual file is much more efficient: when a record is created, peers in the network will only download the newly-created file.

#### Linkability

Putting each record in an individual file also makes each record linkable! This isn't as important as performance, but it's a nice feature to have. See Dog Legs McBoot's status update as an example:

```
dat://232ac2ce8ad4ed80bd1b6de4cbea7d7b0cad1441fa62312c57a6088394717e41/posts/0jbdviucy.json
```

## Changelog

A quick overview of the notable changes to WebDB:

### 4.1.0

Added "helper tables," which make it possible to track private state and build more sophisticated indexes.

### 4.0.0

Replaced JSON-Schema validation with an open `validate` function. This was done to reduce the output bundle size (by 200kb!) and to improve overall flexibility (JSON-Schema and JSON-LD do not work together very well).

### 3.0.0

The `addSource()` and `removeSource()` methods were replaced with `indexArchive()`, `indexFile()`, `unindexArchive()`, and `unindexFile()`.
The `indexArchive()` method also provides an option to disable watching.

This change was made as we found controlling the index was an important part of using managewell.github.io.
Frequently we'd want to index an archive temporarily, for instance to view a user's profile on first visit.

This new API gives better control for those use-cases, and no longer assumes you want to continue watching an archive after indexing it once.
