# react-native-webview-file-upload-android
ReactNative's WebView on Android does not support file input. This library
adds in an `<AndroidWebView>` that does support file input.

*PLEASE READ THE LIMITATIONS SECTION FIRST*

The lack of support appears to be due to Android-level limitations -
specifically that there is no API available for this prior to Android 5 (except
kind of between 4 and 4.4.1 possibly, hard to read from discussions? And using
undocumented APIs prior? Very spotty APIs anyway, and not built-in to WebView)

<!-- MarkdownTOC -->

- [Limitations](#limitations)
- [Further background reading](#further-background-reading)
- [Requirements](#requirements)
- [Installation](#installation)
  - [Manual Linking](#manual-linking)
  - [Automatic Linking](#automatic-linking)
- [Usage](#usage)
- [Contributing](#contributing)

<!-- /MarkdownTOC -->

## Limitations
- This is untested on most Android platforms at time of writing. This was tested
in an AVD with following config: Nexus 5X API 25 x86, Android 7.1.1.

- At present this library handles only uploading images picked from the gallery.

  This could likely be modified to support other file types, or to have generic
  picking from file system instead, but at present I don't have time to make
  these modifications - this rework of the original repo
  (https://github.com/dongyaQin/react-native-webview-file-upload) is for a
  specific project requiring only image uploads.

  Modifications to support uploading other types would need to be made in
  AndroidWebViewManager.java, specifically - you would like want to work out how
  to check the type on the `<input>` element, and pop appropriate intents
  against the `createChooser` call, depending on the file type.

  Such work would likely lean more heavily on the code posted in this comment
  https://github.com/facebook/react-native/issues/5219#issuecomment-209926304.
  You may also want to refer to other threads in the background reading section
  below.

- This only works with Android - specifically, it DOES NOT defer to react-native
built-in WebView for iOS. If supporting both platforms, you will need to include
platform-specific code, using `AndroidWebView` only in iOS (see
https://facebook.github.io/react-native/docs/platform-specific-code.html). For
Image Upload from Gallery in iOS however, this is as simple as ensuring your
`info.plist` contains a `NSPhotoLibraryUsageDescription` entry.

## Further background reading
The following threads provide some further technical background as to the need
for this library, and implementation. These may be of use if modifying this
library.
- https://github.com/facebook/react-native/pull/12807
- https://github.com/facebook/react-native/issues/11230
- https://github.com/facebook/react-native/issues/5219

## Requirements
This has been tested with (and has a peerDependency in package.json on
`react-native` `^0.40.0`). It _might_ work with earlier versions. If it does,
please feel free to open a PR amending the peerDependencies in package.json.

## Installation

Install the library into your project
`npm install react-native-webview-file-upload-android --save`

### Manual Linking

Add the library to the end of the `dependencies` section of your
`android/app/build.gradle`:

```diff
 dependencies {
     //...
+    compile project(':react-native-webview-file-upload-android')
 }
```

Add the library to your `android/settings.gradle`. Don't clobber any other third
party libraries you might be including though! Example of adding to a file with
no other libraries:
```diff
 rootProject.name = 'YourAppName'

-include ':app'
+include ':app', ':react-native-webview-file-upload-android'
+project(':react-native-webview-file-upload-android').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-webview-file-upload-android/android')
```


Add the library's React Package to your `MainApplication.java` (location will
differ from project to project):
```diff
 //...
+import com.oblongmana.webviewfileuploadandroid.AndroidWebViewPackage;

 //...

 public class MainApplication extends Application implements ReactApplication {
     //...
     @Override
     protected List<ReactPackage> getPackages() {
         return Arrays.<ReactPackage>asList(
             new MainReactPackage(),
+            new AndroidWebViewPackage()
         );
     }
     //...
 }
```
### Automatic Linking
Automatic linking as not been tested, so no guarantees that will work. If you
want to try it, use:
```shell
react-native link react-native-webview-file-upload-android
```
And if that does work, please feel free to edit this README!

## Usage
Simply use the `<AndroidWebView>` anywhere you would usually use a `<Webview>`.
As mentioned above, if you are support bothing iOS and Android, you will need
platform-specific code, as `<AndroidWebView>` does not support iOS.

As noted above in the Limitations section, this library currently only
allows for image input, but could be modified to accept other file inputs. As
such, only the image capture will work. If you modify the library to accept
other inputs, I encourage you to submit a PR!

e.g.
Just to reiterate, note that the source URI below contains tests for multiple
types of media capture, but only image capture is presently supported using
this library. See the Usage notes above, and the Limitations section.
```jsx
import React, { Component} from 'react';
import { WebView, View, Platform } from 'react-native';
import AndroidWebView from 'react-native-webview-file-upload-android';
//...
class MySuperSpecialWebView extends Component {
  render() {
    return (
      <View style={{flex:1}}>
        {Platform.select({
              android:  () => <AndroidWebView
                                source={{ uri: 'https://mobilehtml5.org/ts/?id=23' }}
                              />,
              ios:      () => <WebView
                                source={{ uri: 'https://mobilehtml5.org/ts/?id=23' }}
                              />
        })()}
      </View>
    );
  }
}
```

## Contributing
As noted enthusiastically above, contributions are very welcome, especially any
contributions which add support for files other than images selected from the
gallery

This project includes an `.eslintrc.json` file, and any contributions should
comply with this. My personal dev environment was Sublime Text 3, using
- https://github.com/SublimeLinter/SublimeLinter3
- https://github.com/roadhump/SublimeLinter-eslint
- https://github.com/babel/babel-sublime

While developing locally, I would suggest you have a test react-native project
which has a local copy of this library as a dependency. You can accomplish this
by adding
`"react-native-webview-file-upload-android": "file:../react-native-webview-file-upload-android"`
to your `package.json` `dependencies` (obviously, alter the `file:` url to
point to your local copy).
