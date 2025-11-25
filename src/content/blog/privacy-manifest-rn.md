---
title: "Apple privacy manifest for React Native"
description: "A guide on how to setup the Apple privacy manifest for React Native apps"
pubDate: "Apr 22 2024"
heroImage: "/privacy-manifest.png"
---

If you've been shipping to the app store in the past month you will probably have seen this warning from apple

![email from apple](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/avxu93bv81udvr70w7y4.png)

This looks scary and confusing but its actually a lot simpler than it looks.

## What those docs mean

If you follow one of the [links](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_use_of_required_reason_api) in the email it will take you to a wall of text about the `required reason API`.

All this means is that now for apis that could be used to identify a user you must provide a reason for its use.

In the documentation [here](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_use_of_required_reason_api) you will find the different apis that match this description.

For example for file access these are the apis that would need a reason

```
creationDate
modificationDate
fileModificationDate
contentModificationDateKey
creationDateKey
getattrlist(_:_:_:_:_:)
getattrlistbulk(_:_:_:_:_:)
fgetattrlist(_:_:_:_:_:)
stat
fstat(_:_:)
fstatat(_:_:_:_:)
lstat(_:_:)
getattrlistat(_:_:_:_:_:_:)
```

Heres where you can see that in the docs

![image showing this in the docs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5uantoxkyyo82jkwzk1z.png)

## What to do

For most people though you probably aren't using any of these directly and you just need to add reasons for the apis that React Native uses internally.

A member of the community already found out the reasons required and shared them here

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Gist: <a href="https://t.co/litsPv36ez">https://t.co/litsPv36ez</a></p>&mdash; Catalin Miron - AnimateReactNative.com (@mironcatalin) <a href="https://twitter.com/mironcatalin/status/1780335853794951423?ref_src=twsrc%5Etfw">April 16, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

If you're using expo you can update to 50.0.17 and add this to your app.json (or equivalent app.config.js etc)

```json
"ios": {
  "privacyManifests": {
    "NSPrivacyAccessedAPITypes": [
      {
        "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategoryFileTimestamp",
        "NSPrivacyAccessedAPITypeReasons": [
          "C617.1"
        ]
      },
      {
        "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategorySystemBootTime",
        "NSPrivacyAccessedAPITypeReasons": [
          "35F9.1"
        ]
      },
      {
        "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategoryDiskSpace",
        "NSPrivacyAccessedAPITypeReasons": [
          "E174.1"
        ]
      },
      {
        "NSPrivacyAccessedAPIType": "NSPrivacyAccessedAPICategoryUserDefaults",
        "NSPrivacyAccessedAPITypeReasons": [
          "CA92.1"
        ]
      }
    ]
  }
}
```

Check the gist in the tweet above for the full example.

There is also a guide from expo on how to do this in case you're still stuck https://docs.expo.dev/guides/apple-privacy/

### Expo pre v50

For earlier versions of expo check out this comment from [@getKonstantin](https://twitter.com/getKonstantin):

github https://github.com/expo/expo/issues/27796#issuecomment-2059966033

## RN CLI

If you're not using expo then you can follow the steps to create a privacy manifest in XCode.

1. Open your ios project in Xcode
2. Choose File > New File.
3. Scroll down to the Resource section, and select App Privacy File type.
4. Click Next.
5. Check your app target in the Targets list.
6. Click Create.
7. under NSPrivacyAccessedAPITypes add the apis and reasons from above

Your `PrivacyInfo.xcprivacy` file should get something like this

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>NSPrivacyCollectedDataTypes</key>
  <array>
  </array>
  <key>NSPrivacyAccessedAPITypes</key>
  <array>
    <dict>
      <key>NSPrivacyAccessedAPIType</key>
      <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
      <key>NSPrivacyAccessedAPITypeReasons</key>
      <array>
        <string>C617.1</string>
      </array>
    </dict>
    <dict>
      <key>NSPrivacyAccessedAPIType</key>
      <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
      <key>NSPrivacyAccessedAPITypeReasons</key>
      <array>
        <string>CA92.1</string>
      </array>
    </dict>
    <dict>
      <key>NSPrivacyAccessedAPIType</key>
      <string>NSPrivacyAccessedAPICategorySystemBootTime</string>
      <key>NSPrivacyAccessedAPITypeReasons</key>
      <array>
        <string>35F9.1</string>
      </array>
    </dict>
  <dict>
      <key>NSPrivacyAccessedAPIType</key>
      <string>NSPrivacyAccessedAPICategoryDiskSpace</string>
      <key>NSPrivacyAccessedAPITypeReasons</key>
      <array>
        <string>E174.1</string>
      </array>
    </dict>
  </array>
  <key>NSPrivacyTracking</key>
  <false/>
</dict>
</plist>
```

This is a modified version of the file from the react [native cli template](https://github.com/facebook/react-native/blob/main/packages/react-native/template/ios/HelloWorld/PrivacyInfo.xcprivacy)

## What do those values mean?

These values come from the apple documentation, heres what they mean

### File access time

**API Name**: NSPrivacyAccessedAPICategoryFileTimestamp

**C617.1**: Declare this reason to access the timestamps, size, or other metadata of files inside the app container, app group container, or the app’s CloudKit container.

### System boot time

**API Name**: NSPrivacyAccessedAPICategorySystemBootTime

**35F9.1**: Declare this reason to access the system boot time in order to measure the amount of time that has elapsed between events that occurred within the app or to perform calculations to enable timers.

### Disk space

**API Name**: NSPrivacyAccessedAPICategoryDiskSpace

**E174.1**: Declare this reason to check whether there is sufficient disk space to write files, or to check whether the disk space is low so that the app can delete files when the disk space is low. The app must behave differently based on disk space in a way that is observable to users.

### User defaults

**API Name**: NSPrivacyAccessedAPICategoryUserDefaults

**CA92.1**: Declare this reason to access user defaults to read and write information that is only accessible to the app itself.

## Summary

Hopefully you should now be able to get your application warning free, but do make sure you check these values for yourself since you may be using other apis.

Let me know if this works for you or if I got anything wrong here.
