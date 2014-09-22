# Chapter 9: Data Management, Storage and Offline



Maintaining and storing application data was traditionally a heavy lift where the server took on much of the responsibility. With advancements in client-side JavaScript, and the rise of MVC frameworks, we've been better able to handle the implicit model data we encounter on the client-side.

However, developing for mobile devices adds some additional challenges. Mobile users are likely going to be travelling in and out of connected areas, which means we must have a solid strategy for handling offline data. Luckily, there are several tools available to  help us with our client-side storage and offline data.

## Common Strategies
The main goal of offline technologies is to seamlessly store and sync data regardless of internet connectivity. When implementing this functionality, you're likely to notice that offline techniques also benefit other aspects of your application.

For instance, the load on the server is lessened, making your app more responsive and better equipped to handle a robust user experience. Additionally, potential security risks are alleviated as our need to rely on technologies such as cookies and http diminishes.

Below are some of the most common techniques for successful data management in online and offline circumstances. Rec Room apps specifically make use of IndexedDB and Ember Data, but we'll also touch on other popular tools.

### IndexedDB
[IndexedDB][indexed-db] provides client-side storage of structured data, which means we can persistently store data inside a user's browser. It is currently the underlying persistence mechanism in Rec Room apps.

The concepts behind IndexedDB are slightly different than you may be used to when working with a traditional database. IndexedDB is transactional - meaning you are not synchronously storing or retrieving values from the database, rather you are requesting that a database operation occurs. This helps prevent multiple data modifications from overriding each other.

The API for IndexedDB is asynchronous, meaning any requested data will be delivered to a callback, rather than directly through return values.

Adding an index to our object store makes it efficient to query and iterate across, giving us the ability to work on and offline.

With IndexedDB, after opening a database connection, you can create object stores for your application models. For example, in our High Fidelity application from Chapter 4, we would create an object store for our `podcast` model called `podcastObjectStore`. In this object store, we can persist any data for our podcasts as regular JavaScript objects.

````
var request = indexedDB.open("hifi"); // open database named "hifi"

// This handler is called when we are opening a new version of
// our database and allows us to specify an updated schema. 
request.onupgradeneeded = function(event) {
  var db = event.target.result;

  // Create an objectStore to hold information about our podcasts.
  // We're going to use "rssURL" as our unique key path.
  var objectStore = db.createObjectStore("podcasts", { keyPath: "rssURL" });

  // Create an index to search podcasts by name. We may have duplicates
  // so we can't use a unique index.
  objectStore.createIndex("name", "name", { unique: false });

  // Use transaction oncomplete to make sure the objectStore creation is 
  // finished before adding data into it.
  objectStore.transaction.oncomplete = function(event) {
    // Store values in the newly created objectStore.
    var podcastObjectStore = db.transaction("podcasts", "readwrite").objectStore("podcasts");
    for (var i in podcastData) {
      podcastObjectStore.add(podcastData[i]);
    }
  }
};
````

*Note: This is example code. The High Fidelity application utilizes additional technologies (explained later in this section) that hook into IndexedDB.*

IndexedDB is still new and thus may not work consistently across all browsers. For updated information on browser compatibility, check [Can I Use IndexedDB](http://caniuse.com/#feat=indexeddb).

For more information on how to work with IndexedDB, see [Using IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB). There is also a simple [tutorial](http://www.html5rocks.com/en/tutorials/indexeddb/todo/) on [HTML5Rocks](http://www.html5rocks.com) by Paul Kinlan going over basic usage of IndexedDB.


### LocalForage
[localForage][local-forage] is a JavaScript library for asynchronous storage (via IndexedDB or WebSQL where available) with a simple, localStorage-like API.

** code ** 


### Ember Data
Recroom apps include the Ember framework to help construct an MVC architecture. The modularity Ember enforces makes it easy to keep your app data separate from other parts of the application. This allows for greater flexibility when iterating on your application.

The [ember-data][ember-data] library will help us manage data for the Ember models in our application, but first we need to define them.

In our High Fidelity app from Chapter 4, we work with two different models: podcasts and episodes. Let's look at the `podcast` model defined in /app/scripts/models/podcast_model.js:

````
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
  
  `episodes: DS.hasMany('episode', {async: true}) // a podcast model has many episodes`

The `{async: true}` hash allows for [asynchronous data retrieval](http://www.toptal.com/emberjs/a-thorough-guide-to-ember-data#associationModifiers). This means we can request data that has an association or relationship to another data record, and specify a callback to be invoked once our associated data is available.


Ember Data is not tied to IndexedDB by default, (it is agnostic to underlying technologies), so we have included the npm package [ember-indexeddb-adapter](https://github.com/kurko/ember-indexeddb-adapter/) to help persist our data.

````
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

