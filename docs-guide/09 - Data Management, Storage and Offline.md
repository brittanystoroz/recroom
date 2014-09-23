# Chapter 9: Data Management, Storage and Offline

Maintaining and storing application data was traditionally a heavy lift where the server took on much of the responsibility. With advancements in client-side JavaScript, and the rise of MVC frameworks, we've been better able to handle the implicit model data we encounter on the client-side.

However, developing for mobile devices adds some additional challenges. Mobile users are likely going to be travelling in and out of connected areas, which means we must have a solid strategy for handling offline data. Luckily, there are several tools available to  help us with our client-side storage and offline data.

## Common Strategies
The main goal of offline technologies is to seamlessly store and sync data regardless of internet connectivity. When implementing this functionality, you're likely to notice that offline techniques also benefit other aspects of your application.

For instance, the load on the server is lessened, making your app more responsive and better equipped to handle a robust user experience. Additionally, potential security risks are alleviated as our need to rely on technologies such as cookies and http diminishes.

Below are some of the most common techniques for successful data management in online and offline circumstances. Rec Room apps specifically make use of IndexedDB and Ember Data, but we'll also touch on other popular tools.

### IndexedDB
[IndexedDB][indexed-db] provides client-side storage of structured data, which means we can persistently store large amounts of data inside a user's browser. It is currently the underlying persistence mechanism in Rec Room apps.

Rec Room apps utilize additional libraries (explained later in this chapter) that abstract the IndexedDB implementation, but using IndexedDB on it's own is fairly simple. Let's walk through a simple implementation using our High Fidelity app from Chapter 4 as an example.

First you must request to open a database:

```javascript
var request = indexedDB.open("hifi"); // open database named "hifi"
````

Making requests (as we just requested to open a database) is a key concept of IndexedDB. IndexedDB is transactional, meaning rather than directly storing or receiving values from the database, we are requesting that a database operation occurs. This helps prevent multiple data modifications from overriding each other.

When you create a new database, an `onupgradeneeded` event will be triggered. In the handler for this event, we can create **object stores** for our application models. Think of object stores as IndexedDB's equivalent of tables in relational databases.

In our High Fidelity app, we want to create an object store for our `podcast` model. With this object store, we will be able to create records of data for each of our podcasts, and persist them to the database as regular JavaScript objects:

```javascript
request.onupgradeneeded = function(event) {
  var db = event.target.result;

  // Create an objectStore to hold information about our podcasts.
  // We're going to use "rssURL" as our unique key path.
  var objectStore = db.createObjectStore("podcasts", { keyPath: "rssURL" });
};
````

We now have a place to store all of our podcast data, but we'll also want to query this data. With IndexedDB, we can add an **index** to our object store which will make it efficient to query and iterate across.  In the same `onupgradeneeded` handler, we can add an index like so:

```javascript
// Create an index to search podcasts by name. We will set unique
// to false, in case there happen to be podcasts by the same name.
objectStore.createIndex("name", "name", { unique: false });
````

So far we've requested that a podcasts object store be created, and we've requested an index to search it. These interactions are happening as **transactions** - operations for accessing or modifying data in the database. We must wait for these transactions to be completed before we can add data to our object store. In other words, we need to make sure the object store has, in fact, been created.

IndexedDB provides a `transaction.oncomplete` handler that will be called when our object store is ready to accept data:

```javascript
objectStore.transaction.oncomplete = function(event) {
  // Store values in our podcasts object store
  var podcastObjectStore = db.transaction("podcasts", "readwrite").objectStore("podcasts");
  for (var i in podcastData) {
    podcastObjectStore.add(podcastData[i]);
  }
}
````

Now that we have data in our object store, we can retrieve it with a simple `get()`, where we provide a value for the keyPath we specified earlier (`rssUrl`):

```javascript
var transaction = db.transaction(["podcasts"]); // create a new transaction, specifying a list of object stores we will need access to
var objectStore = transaction.objectStore("podcasts"); // get the podcasts object store from our transation
var request = objectStore.get("rss.url.com/podcast-title"); // get a podcast with an rssUrl of rss.url.com/podcast-title
request.onsuccess = function(event) {
  // We are provided with request.result in our onsuccess callback,
  // which gives us access to the property values on our data record
  console.log("Podcast name is: " + request.result.name);
};
````

For more information on how to work with IndexedDB, see [Using IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB) and [Basic Concepts of IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Basic_Concepts_Behind_IndexedDB#gloss_transaction). There is also a simple [tutorial](http://www.html5rocks.com/en/tutorials/indexeddb/todo/) on [HTML5Rocks](http://www.html5rocks.com) by Paul Kinlan going over basic usage of IndexedDB.

IndexedDB is still new and thus may not work consistently across all browsers. The next technology we'll go over, localForage, will help you handle these browser compatibility issues. For updated information on support for IndexedDB, check [Can I Use IndexedDB](http://caniuse.com/#feat=indexeddb).


### LocalForage
[localForage][local-forage] is a JavaScript library that provides a localStorage-like API for interacting with a number of underlying storage technologies.

Callbacks or promises: 

```javascript
function doSomethingElse(value) {
    console.log(value);
}

localforage.setItem('key', 'value', doSomethingElse);
localforage.setItem('key', 'value').then(doSomethingElse);

localforage.getItem('key', alert);
````

localForage will use whichever persistence mechanism is available, in this order:

1. IndexedDB
2. WebSQL
3. localStorage

More documentation is available [here](http://mozilla.github.io/localForage/).


### Ember Data
Recroom apps include the Ember framework to help construct an MVC architecture. The modularity Ember enforces makes it easy to keep your app data separate from other parts of the application. This allows for greater flexibility when iterating on your application.

The [ember-data][ember-data] library will help us manage data for the Ember models in our application, but first we need to define them.

In our High Fidelity app from Chapter 4, we work with two different models: podcasts and episodes. Let's look at the `podcast` model defined in /app/scripts/models/podcast_model.js:

```javascript
HighFidelity.Podcast = DS.Model.extend({
    title: DS.attr('string'),
    description: DS.attr('string'),
    episodes: DS.hasMany('episode', {async: true}), // set up relationship between podcast and episode models
    rssURL: DS.attr('string'),
    lastUpdated: DS.attr('number'),
    lastPlayed: DS.attr('number'),
    coverImageBlob: DS.attr('string'),
    coverImageURL: DS.attr('string')
});
````

Here we are defining our schema and setting up relationships for our model. We declare new attributes on our model with `DS.attr('type')`, where `type` is the value's expected data type. 

Ember Data also provides several relationship types to help you describe how your models associate with each other. In this scenario, we are defining a one-to-many relationship, where a single `podcast` has many `episodes`:
  
```javascript
episodes: DS.hasMany('episode', {async: true}) // a podcast model has many episodes
````

The `{async: true}` hash allows for [asynchronous data retrieval](http://www.toptal.com/emberjs/a-thorough-guide-to-ember-data#associationModifiers). This means we can request data that has an association or relationship to another data record, and specify a callback to be invoked once our associated data is available.


Ember Data is not tied to IndexedDB by default, (it is agnostic to underlying technologies), so we have included the npm package [ember-indexeddb-adapter](https://github.com/kurko/ember-indexeddb-adapter/) to help persist our data.

```javascript
HighFidelity.ApplicationSerializer = DS.IndexedDBSerializer.extend();
HighFidelity.ApplicationAdapter = DS.IndexedDBAdapter.extend({
    //autoIncrement: true,
    databaseName: 'hifi',
    version: 1,
    migrations: function() {
        this.addModel('podcast');
        this.addModel('episode');
    }
});
````

## Offline First
Empathy for offline users is hampered by the high-speed internet we're likely connected to during development. To truly build with offline users in mind, develop your application as if it will never have an internet connection. See how your application looks on a device or in a simulator with wifi and cellular data turned off. 

- What UI will the user see? **[packaged apps save HTML/CSS/JS by default]**
- What interactivity is still available to them?
- Will their actions offline be reflected next time they are reconnected?

Finally, if your app (or a significant part of your app) does not work offline, be sure to indicate that to your users. **[ do a check for online/offline connectivity, code example ]**

For more guidance on offline development, see our [Offline apps developer recommendations page](https://developer.mozilla.org/en-US/Apps/Build/Offline).



[indexed-db]: https://www.google.com/url?q=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FIndexedDB&sa=D&sntz=1&usg=AFQjCNHuwdSJ3rzZYhJSoAA6UrMuLW0Bvg
[local-forage]: https://github.com/mozilla/localForage
[ember-data]: https://github.com/emberjs/data

