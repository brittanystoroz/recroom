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
[localForage][local-forage] is a JavaScript library that provides a localStorage-like API for interacting with a number of underlying storage technologies. It can be added to your application as a simple JavaScript file, or be installed via bower:

`bower install localforage`

As mentioned in the previous section, IndexedDB is not consistently supported across all browsers. localForage hides these types of inconsistencies from the user by implementing fallbacks when a backend driver for the datastore (such as IndexedDB) is not available. By default, localForage will select drivers in this order:

1. IndexedDB
2. WebSQL
3. localStorage

With browser compatibility issues out of your way, you're free to focus on storing and retrieving your application data.

The API for localForage is asynchronous: instead of our data being delivered through return values, it will be sent to a defined callback. With this implementation, setting and getting data with localforage will look like this:

```javascript
// Set an item and specify a callback
localforage.setItem('key', 'value', callbackYouDefine);

// Get an item and specify a callback that alerts the value
localforage.getItem('key', function(value) {
  console.log('Retrieved value is: ' + value);
})
````

You can also use promises rather than callbacks if you prefer:

```javascript
function callbackYouDefine(value) {
    console.log(value);
}

localforage.setItem('key', 'value').then(callbackYouDefine);
````

More documentation is available [here](http://mozilla.github.io/localForage/).


### Ember Data
Recroom apps include the Ember framework to help construct an MVC architecture. The modularity Ember enforces makes it easy to keep your app data separate from other parts of the application. This allows for greater flexibility when iterating on your application.

The [ember-data][ember-data] library will help us manage data for the Ember models in our application, but first we need to define them.

In our High Fidelity app from Chapter 4, we work with two different models: podcasts and episodes. Let's look at the `podcast` model defined in /app/scripts/models/podcast_model.js:

```javascript
HighFidelity.Podcast = DS.Model.extend({
    title: DS.attr('string'),
    description: DS.attr('string'),
    episodes: DS.hasMany('episode', {async: true}),
    rssURL: DS.attr('string'),
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
Empathy for offline users is hampered by the high-speed internet we're likely connected to during development. To truly build with offline users in mind, develop your application as if it will never have an internet connection. See how your application looks on a device or in a simulator with wifi and cellular data turned off. It may be helpful to ask yourself the following questions while you're developing for offline use:

- What UI will the user see?
- What interactivity is still available to them?
- Will their actions offline be reflected next time they are reconnected?

### Detecting Connectivity
In order to provide a more seamless user experience regardless of connectivity, we must be able to determine the status of our user's connection. If a significant part of your application does not work offline, you'll want to make sure to indicate that to your users.

[offline.js](http://github.hubspot.com/offline/docs/welcome/) is an offline detection library that makes it easy to confirm the state of a user's connectivity, and gracefully handle any events or tasks that need to occur when toggling between on and offline. This could mean updating data, or simply displaying more informative messaging.

You can start working with offline.js by simply adding it as a JavaScript file to your page.

offline.js tests connectivity by making an XHR request to load a `/favicon.ico` file by default. You can configure the file it checks for by providing a different URL in `Offline.options`:

```javascript
Offline.options = {checks: {xhr: {url: '/connection-test'}}};
````

Once our connectivity check is configured, we can tell offline.js to detect the current state by calling `Offline.check()`. This will return an XHR response, and sets `Offline.state` to either `up` (if we are online) or `down` (if we are offline).

We can now get the user's state by asking for the value of `Offline.state`, and provide callbacks to execute during different stages of connectivity. offline.js provides several helpful events that we can bind to for handling state changes. For example, if we want to display a notification to the user that they have gone offline, we can simply call:

```javascript
Offline.on('down', function() {
  var message = "You can continue listening to your saved podcasts, but cannot search for new ones until a connection returns.";
  var offlineNotification = new Notification('You Are Offline', { body: message });
});
````

### When to Save State
A solid, offline-first application will store its assets and data offline on its first load after installation. This will ensure that there are available resources for your app to pull from during subsequent uses when offline. When a connection is regained, the application should synchronize any data and update available assets.

During app usage, it is a good idea to periodically save the user's state to your offline data store. Additionally, you will want to save the state locally whenever a user closes your application so they can pick up where they left off next time they open it.

For more guidance on offline development, see our [Offline apps developer recommendations page](https://developer.mozilla.org/en-US/Apps/Build/Offline).


[indexed-db]: https://www.google.com/url?q=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FIndexedDB&sa=D&sntz=1&usg=AFQjCNHuwdSJ3rzZYhJSoAA6UrMuLW0Bvg
[local-forage]: https://github.com/mozilla/localForage
[ember-data]: https://github.com/emberjs/data

