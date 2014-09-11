# Rec Room Documentation

![Rec Room logo](images/recroom-logo.jpg?raw=true)


# Chapter 9: Data Management, Storage and Offline

## why/benes/random notes
  - offline techniques lighten the load on servers, making your app more responsive
  - eases security risks by lessening our need to rely on cookies/http
  - increases performance, opening up new opps for better UX
  - if your app does not work offline, make sure there is an indicator notifying users of that
  - users may close your app at any time: save their state to allow them to pick up where they left off

## Offline First
  - spend a lot of time developing from the comfort of our high-speed internet, hampering our empathy for our offline users
  - easy to take our connection to and from the server for granted, only when you start developing for offline do you realize how reliant we are on that relationship
  - being offline is/was/has been *the* selling point of mobile devices, while we're getting more connected, developing for offline still needs to be a top priority
  - write your app as if it has no internet connection.
  - [Offline apps developer recommendations page](https://developer.mozilla.org/en-US/Apps/Build/Offline)

There are several technologies available that make data management and storage easier. Our starter app from [Chapter 4](04 - Let's Build a Rec Room App.md) makes use of the following (does it? I actually haven't looked enough):

## IndexedDB
[IndexedDB][indexed-db] provides client-side storage of structured data with support for high-performance searches.

## LocalForage
[localForage][local-forage] is a JavaScript library for asynchronous storage (via IndexedDB or WebSQL where available) with a simple, localStorage-like API.

## Ember Data
  - using an MVC framework like ember makes it super easy to keep your app data separate from other parts of the application
  - the ember data library helps manage model data; still in beta but API is pretty stable
  - [ember-data][ember-data]

TODO code example from high fidelity using storage technologies

[indexed-db]: https://www.google.com/url?q=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FIndexedDB&sa=D&sntz=1&usg=AFQjCNHuwdSJ3rzZYhJSoAA6UrMuLW0Bvg
[local-forage]: https://github.com/mozilla/localForage
[ember-data]: https://github.com/emberjs/data

