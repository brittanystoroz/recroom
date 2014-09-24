# Rec Room Documentation

![Rec Room logo](images/recroom-logo.jpg?raw=true)


# Chapter 8: Web APIs - Accessing Phone APIs and Hardware

WebAPIs are simply JavaScript APIs that allow applications to interact with mobile device features. There are a number of WebAPIs available for accessing device hardware (battery status, device vibration hardware, etc.) and data (calendar data, contacts, etc.). Leveraging these will help you build a feature-rich application.

In Chapter 7 we learned how to set permissions in our application manifest to access these types of APIs. Let's walk through a couple of examples of putting them to work.

From our podcasts app in Chapter 4, let's imagine we want to push a notification to the user's device when a new episode of a podcast is finished downloading. In our application manifest, we'll need to add the permission `desktop-notification`:

```javascript
"permissions": {
    "desktop-notification": {
      "description": "Needed for creating system notifications."
    }
  }
````

If your application tries to use a WebAPI without being declared in the permissions field of your app manifest, that code will fail to execute.



Specifically, our application requested permission to the `audio-channel-content`, `storage`, and `systemXHR` APIs. Let's take a look at how these were used.

Setting `storage` in our permissions simply allows us to utilize data storage in IndexedDB without size limitations. Besides adding storage to our permissions, there is no additional code necessary to work with this particular API.

The `systemXHR` permission allows anonymous cross-origin XHR without the site having CORS enabled. This is necessary for us to request and fetch podcast feeds.

And finally, `audio-channel-content` specifies that our podcast audio will play on a 'content' audio channel. Audio channels determine what sounds should be prioritized when using multiple applications or device features with audio. See [WebAPI/AudioChannels](https://wiki.mozilla.org/WebAPI/AudioChannels) for additional information.

For a list of the APIs that can be accessed with hosted and privileged apps, see MDN's [App Permissions](https://developer.mozilla.org/en-US/Apps/Build/App_permissions) table.  For more information on working with these APIs, see the [WebAPI](https://developer.mozilla.org/en-US/docs/WebAPI) page.

