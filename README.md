# Integrating Unity Project into Xcode Swift Project

This repo contains a demo Xcode 9.1 iOS project (Swift 4.0) that builds a single view app with an embeded Unity scene
(Unity 2017.2.0f3) with two-way communication between Unity and Swift.  It contains automation script that could be
easily re-used in existing Unity and Swift projects.

![Demo Video](https://github.com/jiulongw/swift-unity/raw/master/images/demo.gif)

This would not be possible without [tutorial by BLITZ][1] and [video by the-nerd Frederik Jacques][2].  Several updates
are applied to fit latest Xcode and Unity releases.

## Compatibility
### Unity
* 2017.1.1
* 2017.1.2
* 2017.2.0
* 2017.3.0
* 2017.3.1
* 2017.4.1
* 2017.4.2

### Xcode
* Xcode 9.0
* Xcode 9.1
* Xcode 9.2

## FAQ
* [Exports.xcconfig file missing][6]
* [Xcode compile error when using .Net 4.6 backend in Unity][9]
* [Entry point (`_main`) undefined][10]

## Why did I create this
I like to automate things when I find myself doing the same thing several times, and potentially over and over again.
If you are (like I am) tired of these tedious steps every time you export a new Unity build, you will love this
automation.  Once integrated, Swift project gets updated automatically everytime a new Unity build is exported.

## How to run the demo
First open Unity project, open `Demo` scene, open `Build Settings` and switch to iOS platform, then hit `Build`.  You
can put output anywhere but better put it to some temporary folder such as `/tmp`.

Once build succeeded, open Xcode project, select target device (Unity does not support `x86_64`) and hit
`Build and then run`.  You might need to change bundle identifier if Xcode has problem creating provisioning profile.

## How to use it in other projects
### Unity
Copy this [post build script][5] to your Unity project's `Assets` folder.  Put anywhere you like but since this is an
editor script, Unity requires it to be under a folder named `Editor`.

Open the script and follow the instructions in the comments.  You need to tell it the path to your Swift project and
the name of your awesome product.

```cs
/// <summary>
/// Path to the root directory of Xcode project.
/// This should point to the directory of '${XcodeProjectName}.xcodeproj'.
/// It is recommended to use relative path here.
/// Current directory is the root directory of this Unity project, i.e. the directory of 'Assets' folder.
/// Sample value: "../xcode"
/// </summary>
private const string XcodeProjectRoot = <PROJECT PATH>;

/// <summary>
/// Name of the Xcode project.
/// This script looks for '${XcodeProjectName} + ".xcodeproj"' under '${XcodeProjectRoot}'.
/// Sample value: "DemoApp"
/// </summary>
private const string XcodeProjectName = <PROJECT NAME>;
```

You're done with Unity part!

### Xcode
Xcode side requires a bit more *one time* effort.

1. Copy entire [Unity][7] folder to the root of your Swift project and add it to Xcode source tree. Make sure all the
`.mm` files are added to correct build target.

![Xcode source tree](https://github.com/jiulongw/swift-unity/raw/master/images/xcode_source_tree.png)

2. Add following lines to the .gitignore file in the same directory of your `.xcodeproj` entry. If you don't have one
there, create one or merge it to your higher level git ignore file with proper updates to the paths.

```
# This file is generated by Unity post build action
DemoApp/Unity/Exports.xcconfig

# Files under Libraries will be copied from Unity build during Xcode build
DemoApp/Unity/Libraries/

# Files under Classes will be copied from Unity build during Xcode build
DemoApp/Unity/Classes/
```

3. Select `Unity` as project configuration file. If you already have your own, merge `Unity/Unity.xcconfig` to your own.

![Unity configuration file](https://github.com/jiulongw/swift-unity/raw/master/images/unity_configuration_file.png)

4. If you don't have Objective-C bridging header file, you can skip this.  Otherwise, merge `Unity/Bridging-Header.h`
to your own bridging header and comment out following lines in `Unity/Unity.xcconfig`:
```
SWIFT_OBJC_BRIDGING_HEADER = $(PRODUCT_NAME)/Unity/Bridging-Header.h;
SWIFT_PRECOMPILE_BRIDGING_HEADER = YES;
```

5. Add following build scripts to your Xcode build target. You can copy from `DemoApp` settings. Note the order. If your project name has space(s) in it, surround the `$PRODUCT_NAME` with double quote.

```sh
echo "Syncing code from $UNITY_IOS_EXPORT_PATH..."
rsync -rc --exclude-from="$PRODUCT_NAME"/Unity/rsync_exclude --delete $UNITY_IOS_EXPORT_PATH/Classes/ "$PRODUCT_NAME"/Unity/Classes/
rsync -rc --exclude-from="$PRODUCT_NAME"/Unity/rsync_exclude --delete $UNITY_IOS_EXPORT_PATH/Libraries/ "$PRODUCT_NAME"/Unity/Libraries/
```
![Xcode build script 1](https://github.com/jiulongw/swift-unity/raw/master/images/xcode_build_script_1.png)


```sh
echo "Syncing data from $UNITY_IOS_EXPORT_PATH..."
rm -rf "$TARGET_BUILD_DIR/$PRODUCT_NAME.app/Data"
cp -Rf "$UNITY_IOS_EXPORT_PATH/Data" "$TARGET_BUILD_DIR/$PRODUCT_NAME.app/Data"
```

![Xcode build script 2](https://github.com/jiulongw/swift-unity/raw/master/images/xcode_build_script_2.png)

6. Update `AppDelegate.swift` to initialize Unity during application start.  Follow the [sample AppDelegate code][4].

7. Once Unity is loaded, a `UnityReady` notification will be triggered. Subscribe to that notification, grab Unity
view or view controller and present them as you like.  Follow the [sample ViewController code][8].
```swift
// Get Unity View
UnityGetGLView()

// Get Unity View Controller
UnityGetGLViewController()
```
Now you should be good to go.  Give it a try!

Note: Building the Unity project for the first time will modify Xcode project file a lot.
It is safe to commit the change into source control as it only performs necessary updates going forward.


## Updates
* 02/19/2018 - Support Unity version 2017.3.1f3.
* 12/22/2017 - Support Unity version 2017.3.0f3.
* 11/15/2017 - Updated demo focusing on reusability. Updated README accordingly.
* 11/2/2017 - Support Unity version from 2017.1.1f1 to 2017.2.0f3.
* 10/25/2017 - Added FAQ about missing Exports.xcconfig file.
* 10/6/2017 - Fix previous known issue that new files generated by Unity are not added to Xcode project automatically.
* 9/26/2017 - Initial commit


[1]: https://github.com/blitzagency/ios-unity5
[2]: http://www.the-nerd.be/2015/08/20/a-better-way-to-integrate-unity3d-within-a-native-ios-application/
[3]: https://developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html
[4]: https://github.com/jiulongw/swift-unity/blob/master/demo/xcode/DemoApp/AppDelegate.swift
[5]: https://github.com/jiulongw/swift-unity/blob/master/XcodePostBuild.cs
[6]: https://github.com/jiulongw/swift-unity/issues/8
[7]: https://github.com/jiulongw/swift-unity/tree/master/demo/xcode/DemoApp/Unity
[8]: https://github.com/jiulongw/swift-unity/blob/master/demo/xcode/DemoApp/ViewController.swift
[9]: https://github.com/jiulongw/swift-unity/issues/31
[10]: https://github.com/jiulongw/swift-unity/issues/17
