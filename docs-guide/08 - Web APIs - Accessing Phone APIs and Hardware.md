# Rec Room Documentation

![Rec Room logo](images/recroom-logo.jpg?raw=true)


# Chapter 8: Web APIs - Accessing Phone APIs and Hardware

WebAPIs are simply JavaScript APIs that allow applications to interact with mobile device features. There are a number of WebAPIs available for accessing device hardware (battery status, device vibration hardware, etc.) and data (calendar data, contacts, etc.). Leveraging these will help you build a feature-rich application.

In Chapter 7 we learned how to set permissions in our application manifest to access these types of APIs. In this chapter we'll walk through a couple examples of putting them to work.

From our podcasts app in Chapter 4, let's imagine we want to push a notification to the user's device when a new episode of a podcast is finished downloading. In our application manifest, we'll need to add the permission `desktop-notification` so we can use the [Notifications WebAPI](https://developer.mozilla.org/en-US/docs/Web/API/Notification/Using_Web_Notifications):

```javascript
"permissions": {
    "desktop-notification": {
      "description": "Needed for creating system notifications."
    }
  }
````

After setting the appropriate permissions, creating a new notification is simple:

```javascript
function notifyOnDownloadComplete() {
  var message = "Your podcast episode has finished downloading!";
  var offlineNotification = new Notification('Download Complete', { body: message });
}
````

But what would happen if we had forgotten to add the `desktop-notification` permission to our app manifest? In this scenario, users just wouldn't receive a notification upon completion of a download. This is a small feature of a much larger application &emdash; and users may not even realize something is broken &emdash; but when a WebAPI governs more substantial functionality, be sure you've set the necessary permissions. If you're confident you are using an API correctly and it still doesn't seem to be working, double-check that you didn't leave anything out of the permissions declaration in your app manifest.

Now let's make sure this notification only displays if a user begins an episode download and switches applications or returns to the home screen before the download completes. If they begin a download, and are still in the application, they will not need a notification because our UI will have a visual indicator that it's complete.

In order to do this, we'll need to detect if the application is visible or not. Using the [Page Visibility API](https://developer.mozilla.org/en-US/docs/Web/Guide/User_experience/Using_the_Page_Visibility_API), we can add an event listener to our document to see if it is still visible or not. 

```javascript
document.addEventListener("visibilitychange", function() {
  if (document.hidden) {
    console.log("App is hidden");
  } else {
    console.log("App is visible");
  } 
});
````

*Note: The PageVisibility API is not a WebAPI that we need to specify a permission for. We get this one for free. =P*

So whenever a user clicks out of our application or returns to their home screen, a `visibilitychange` event is going to be fired on our document. We'll handle creating the notification when `document.hidden` is `true`.

In our episode model (/app/scripts/models/episode_model.js), we are doing a check to see when the download is complete. Upon completion, we are setting the model's `isDownloading` property to false, and setting it's `isDownloaded` property to true. Let's also call our `notifyOnDownloadComplete` function if the document is hidden:

```javascript
if (_this.get('_chunkCount') ===
    _this.get('_chunkCountSaved') &&
    _this.get('_loadComplete')) {
    _this.set('isDownloading', false);
    _this.set('isDownloaded', true);
    if (document.hidden) {
      notifyOnDownloadComplete();
    }
}
````

This will send a notification to the user's device when the download is complete if they have moved our app to the background.


Specifically, our application requested permission to the `audio-channel-content`, `storage`, and `systemXHR` APIs. Let's take a look at how these were used.

Setting `storage` in our permissions simply allows us to utilize data storage in IndexedDB without size limitations. Besides adding storage to our permissions, there is no additional code necessary to work with this particular API.

The `systemXHR` permission allows anonymous cross-origin XHR without the site having CORS enabled. This is necessary for us to request and fetch podcast feeds.

And finally, `audio-channel-content` specifies that our podcast audio will play on a 'content' audio channel. Audio channels determine what sounds should be prioritized when using multiple applications or device features with audio. See [WebAPI/AudioChannels](https://wiki.mozilla.org/WebAPI/AudioChannels) for additional information.

For a list of the APIs that can be accessed with hosted and privileged apps, see MDN's [App Permissions](https://developer.mozilla.org/en-US/Apps/Build/App_permissions) table.  For more information on working with these APIs, see the [WebAPI](https://developer.mozilla.org/en-US/docs/WebAPI) page.

