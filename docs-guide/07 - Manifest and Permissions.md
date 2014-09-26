# Rec Room Documentation

![Rec Room logo](images/recroom-logo.jpg?raw=true)


# Chapter 7: Manifest and Permissions

Every open web app requires a manifest.webapp file to be placed in the app's root folder. The concept of a manifest file is common practice, and you may be familiar with it from building other tools or apps for different environments. The manifest provides important details about your application (name, description, version, icons, locale strings, etc.) and should be readable by both users and the Firefox Marketplace.

The manifest file is also where you will specify permissions for your application that will allow it to provide advanced functionality, and alert users of what your application needs access to.

Let's look at the webapp.manifest file from the podcasts app introduced in Chapter 4:

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

Note that the icon paths are absolute from the app's origin. Paths to externally hosted icons must be fully qualified. Icons must be square, and in `.png` format.

## Application Types
The `type` and `permissions` fields are both important to ensure your application has access to the APIs it needs. Different types of apps grant access to varying levels of permissions. If you are specifying permissions for your application, the type of application must coincide accordingly. The following type values affect permissions accordingly:

`web`: grants least access to WebAPIs, used for apps that do not require any special permissions
`privileged`: greater access to WebAPIs that are available for developers to use when set in the `permissions` field of the app manifest
`certified`: access to APIs that control critical device functions, not generally available for use by third party developers

Your apps will generally have a type of `web` or `privileged`. When you are specifying permissions for your application, you must specify the type as `privileged`. If the `type` field is missing, it will default to `web`, which will cause unexpected behaviors when trying to access APIs.

## Permissions
When setting permissions for your application, each permission should have a name, description and access level.

Each permission requires:

    name: the name of the permission
    description: the reason why your app needs to use this permission
    access: the level of access required (only needed for device storage and contacts APIs)


PODCAST APP EXAMPLE, ADD ADDITIONAL EXAMPLE THAT REQUIRES ACCESS LEVEL FIELD PERMISSION
For example, here's an app that needs permission to use the device's contacts and alarms.

<!-- "permissions": {
  "contacts": {
    "description": "Required for autocompletion in the share screen",
    "access": "readcreate"
    },
  "alarms": {
    "description": "Required to schedule notifications"
  }
} -->



To troubleshoot and validate your manifest file, you can use Mozilla's [app validator][https://marketplace.firefox.com/developers/validator].

[Optional manifest fields][https://developer.mozilla.org/en-US/Apps/Build/Manifest#Optional_App_Manifest_Fields]
[App Permissions Matrix][https://developer.mozilla.org/en-US/Apps/Build/App_permissions]

default_locale
activities
appcache_path
chrome
csp
fullscreen
installs_allowed_from
locales
messages
orientation
origin
permissions
precompile
redirects
role
version

[App CSP](https://developer.mozilla.org/en-US/Apps/CSP)



Specifically, our application requested permission to the `audio-channel-content`, `storage`, and `systemXHR` APIs. Let's take a look at how these were used.

Setting `storage` in our permissions simply allows us to utilize data storage in IndexedDB without size limitations. Besides adding storage to our permissions, there is no additional code necessary to work with this particular API.

The `systemXHR` permission allows anonymous cross-origin XHR without the site having CORS enabled. This is necessary for us to request and fetch podcast feeds.

And finally, `audio-channel-content` specifies that our podcast audio will play on a 'content' audio channel. Audio channels determine what sounds should be prioritized when using multiple applications or device features with audio. See [WebAPI/AudioChannels](https://wiki.mozilla.org/WebAPI/AudioChannels) for additional information.