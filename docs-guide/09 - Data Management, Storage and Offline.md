# Chapter 9: Data Management, Storage and Offline


** blah blah a sentence or two here **

We face an additional set of challenges maintaining our data when developing for mobile devices. Because users may be travelling in and out of connected areas, it's important to have a good strategy around handling offline data.

The main goal of offline technologies is to seamlessly store and sync data regardless of internet connectivity. When implementing this functionality, you're likely to notice that offline techniques also benefit other aspects of your application. For instance, the load on the server is lessened, making your app more responsive and better equipped to handle a robust user experience. Additionally, potential security risks are alleviated as our need to rely on technologies such as cookies and http diminishes. 

** blah blah a sentence or two here **

## Common Strategies
There are several technologies available that make data management and storage easier:

### IndexedDB
[IndexedDB][indexed-db] provides client-side storage of structured data. This means we can persistently store data inside a user's browser. 

- IndexedDB allows your application to work online and offline
- transactional, trying to use a transaction after it has completed throws exceptions, cant stomp all over eachothers modifications
- mostly asynchronous API
- you don't store or retrieve values from the database through synchronous means. instead you request that a db operation happens.
- you are notified by a DOM event that has properties for helping you determine the status of your request and specifying listeners.
- you create an Object Store **what?** for a type of data and simply persist Javascript Objects to that store.
- each object store can have a collection of indexes that make it efficient to query and iterate across (giving us the ability to work on and offline)
- The Asynchronous API is a non-blocking system and as such will not get data through return values, but rather will get data delivered to a defined callback function. 

IndexedDB is currently the underlying persistence mechanism in recroom apps, though we are using some additional libraries that obfuscate this code which will be explained later in this section.
2
For more information on how to work with IndexedDB, see [Using IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB). There is also a simple [tutorial](http://www.html5rocks.com/en/tutorials/indexeddb/todo/) on [HTML5Rocks](http://www.html5rocks.com) by Paul Kinlan going over basic usage of IndexedDB.

** code ** 

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
  
  episodes: DS.hasMany('episode', {async: true}), // a podcast model has many episodes

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

What UI will the user see? **[packaged apps save HTML/CSS/JS by default]** What interactivity is still available to them? Will their actions offline be reflected next time they are reconnected?

Finally, if your app (or a significant part of your app) does not work offline, be sure to indicate that to your users. **[ do a check for online/offline connectivity, code example ]**

For more guidance on offline development, see our [Offline apps developer recommendations page](https://developer.mozilla.org/en-US/Apps/Build/Offline).



[indexed-db]: https://www.google.com/url?q=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FIndexedDB&sa=D&sntz=1&usg=AFQjCNHuwdSJ3rzZYhJSoAA6UrMuLW0Bvg
[local-forage]: https://github.com/mozilla/localForage
[ember-data]: https://github.com/emberjs/data

