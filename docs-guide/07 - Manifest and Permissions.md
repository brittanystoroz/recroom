# Rec Room Documentation

![Rec Room logo](images/recroom-logo.jpg?raw=true)


# Chapter 7: Manifest and Permissions

When we're creating an open web app, our most essential file is going to be our `manifest.webapp`. This is a required manifest file that you'll want to place in the root folder of your application.

The concept of a manifest file is common practice, and you may be familiar with it from building other tools or apps for different environments. The manifest provides important details about your application (name, description, version, icons, etc.) and should be readable by both users and the Firefox Marketplace.

The manifest file is also where you will specify permissions for your application that will allow it to provide advanced functionality, and alert users of what your application needs access to.

Let's look at the `manifest.webapp` file from the podcasts app introduced in Chapter 4:

```
{
    "version": "2.0.0",
    "name": "Podcasts",
    "description": "Listen to your favourite podcasts on your HTML5-powered device or browser.",
    "icons": {
        "32": "/images/icon-32.png",
        "60": "/images/icon-60.png",
        "64": "/images/icon-64.png",
        "90": "/images/icon-90.png",
        "120": "/images/icon-120.png",
        "128": "/images/icon-128.png",
        "256": "/images/icon-256.png",
        "512": "/images/icon-512.png"
    },
    "developer": {
        "name": "Mozilla",
        "url": "https://mozilla.org/"
    },
    "launch_path": "/index.html",
    "installs_allowed_from": ["https://marketplace.firefox.com", "*"],
    "type": "privileged",
    "permissions": {
        "audio-channel-content": {
            "description": "Required to play audio in the background."
        },
        "storage": {
            "description": "Required to store podcast audio files and images."
        },
        "systemXHR": {
            "description": "Required to download podcasts."
        }
    }
}
```

Some of the fields are self-explanatory: you must have a name for your application, as well as a short description and information about the developer or author of the app. While a version number isn't required, it is generally recommended so you can keep track of your application's releases as you iterate on it.

The `icons` field is where you will specify paths to `.png` images for your icons. If your icons are hosted internally within your application, you should use absolute paths from the app's origin. Any external paths should instead be fully qualified. Currently, in order to submit your app to the Firefox Marketplace, you must have one icon at 128x128, though another at 512x512 is recommended.

The `launch_path` field is only required for Packaged Apps - it will be the path that is loaded when the app starts. In our case, it is simply our application's `/index.html` file.

Though not required, the `installs_allowed_from` field will let us specify where users can initiate an installation of our application from. If this field is left blank, or an array containing `["*"]` is specified, installations are allowed from any site. Specifying an empty array `[]` will prevent all installations from any site.

## Application Types
The `type` and `permissions` fields are both important to ensure your application has access to the WebAPIs it needs. WebAPIs allow your app to access mobile device hardware and data, and will be explained further in the following chapter.

Different types of apps grant access to varying levels of permissions. If you are going to be requesting permissions for your application to access WebAPIs, the type of application must coincide accordingly. The following type values affect permissions accordingly:

`web`: grants least access to WebAPIs, used for apps that do not require any special permissions  
`privileged`: greater access to WebAPIs that are available for developers to use when set in the `permissions` field of the app manifest  
`certified`: access to APIs that control critical device functions, not generally available for use by third party developers

Your apps will generally have a type of `web` or `privileged`. When you are specifying permissions for your application, you must specify the type as `privileged`. If the `type` field is missing, it will default to `web`, which will cause unexpected behaviors when trying to access WebAPIs.

## Permissions
When setting permissions for your application, each permission should have a name and description. Some also require an access level to be set.

- name: the name of the permission
- description: the reason why your app needs to use this permission
- access: the level of access required (only needed for device storage and contacts APIs)

From our Podcasts application, the manifest requested permission to the `audio-channel-content`, `storage`, and `systemXHR` APIs:

```javascript
"type": "privileged",
"permissions": {
    "audio-channel-content": {
        "description": "Required to play audio in the background."
    },
    "storage": {
        "description": "Required to store podcast audio files and images."
    },
    "systemXHR": {
        "description": "Required to download podcasts."
    }
}
````

These all govern important functionality for our application.

Setting `storage` in our permissions gives us data storage in IndexedDB without size limitations. The `systemXHR` permission allows anonymous cross-origin XHR without the site having CORS enabled. This is necessary for us to request and fetch podcast feeds.

And finally, `audio-channel-content` specifies that our podcast audio will play on a 'content' audio channel. Audio channels determine what sounds should be prioritized when using multiple applications or device features with audio. See [WebAPI/AudioChannels](https://wiki.mozilla.org/WebAPI/AudioChannels) for additional information.


To troubleshoot and validate your manifest file, you can use Mozilla's [app validator][https://marketplace.firefox.com/developers/validator].

[Optional manifest fields][https://developer.mozilla.org/en-US/Apps/Build/Manifest#Optional_App_Manifest_Fields]
[App Permissions Matrix][https://developer.mozilla.org/en-US/Apps/Build/App_permissions]


[App CSP](https://developer.mozilla.org/en-US/Apps/CSP)



