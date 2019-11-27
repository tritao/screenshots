
[![pub package](https://img.shields.io/pub/v/screenshots.svg)](https://pub.dartlang.org/packages/screenshots) 
[![Build Status](https://travis-ci.com/mmcc007/screenshots.svg?branch=master)](https://travis-ci.com/mmcc007/screenshots)
[![Build status](https://ci.appveyor.com/api/projects/status/b6vlfyio9e21hxw8?svg=true)](https://ci.appveyor.com/project/mmcc007/screenshots)
[![codecov](https://codecov.io/gh/mmcc007/screenshots/branch/master/graph/badge.svg)](https://codecov.io/gh/mmcc007/screenshots)

![alt text][demo]

[demo]: https://i.imgur.com/gkIEQ5y.gif "Screenshot with overlaid 
status bar placed in frame"  
A screenshot image with overlaid status bar placed in a device frame.  

For an example of images generated with _Screenshots_ on a live app in both stores see:  
[![GitErDone](https://play.google.com/intl/en_us/badges/images/badge_new.png)](https://play.google.com/store/apps/details?id=com.orbsoft.todo)
[![GitErDone](https://linkmaker.itunes.apple.com/en-us/badge-lrg.svg?releaseDate=2019-02-15&kind=iossoftware)](https://itunes.apple.com/us/app/giterdone/id1450240301)


See a demo of _Screenshots_ in action:
[![Screenshots demo](https://i.imgur.com/V9VFSYb.png)](https://vimeo.com/317112577 "Screenshots demo - Click to Watch!")

For introduction to _Screenshots_ see https://medium.com/@nocnoc/automated-screenshots-for-flutter-f78be70cd5fd.

# _Screenshots_

_Screenshots_ is a standalone command line utility and package for capturing screenshot images for Flutter.   

_Screenshots_ will start the required android emulators and iOS simulators (or find attached devices), run tests, process the captured screenshots, and drop them off to Fastlane for delivery to both stores.

_Screenshots_ is inspired by three tools from Fastlane:  
1. [Snapshots](https://docs.fastlane.tools/getting-started/ios/screenshots/)  
This is used to capture screenshots on iOS using iOS UI Tests.
1. [Screengrab](https://docs.fastlane.tools/actions/screengrab/)  
This captures screenshots on android using Android Espresso tests.
1. [FrameIt](https://docs.fastlane.tools/actions/frameit/)  
This is used to place captured iOS screenshots in a device frame.

Since all three of these Fastlane tools do not work with Flutter, _Screenshots_ combines key features of these Fastlane tools into one tool. 

# Features
Since Flutter integration testing is designed to work transparently across iOS and Android, capturing images using _Screenshots_ is easy.

Features include: 
1. Works with existing tests  
Add a single line for each screenshot.
1. Run tests on any device  
Select the devices to run on using a convenient config file. _Screenshots_ will find the devices (real or emulated) and run tests.
1. One run for both platforms  
_Screenshots_ runs tests on both iOS and Android in one run.  
(as opposed to making separate Snapshots and Screengrab runs)
1. One run for multiple locales  
If app supports multiple locales, _Screenshots_ will optionally set the locales listed in the config file before running each test.
1. One run for frames  
Optionally places images in device frames in same run.  
(as opposed to making separate FrameIt runs... which supports iOS only)
1. One run for clean status bars  
Every image that _Screenshots_ generates has a clean status bar.  
(no need to run a separate stage to clean-up status bars)
1. Works with Fastlane  
_Screenshots_ drops-off images where Fastlane expects to find them. Fastlane's [deliver](https://docs.fastlane.tools/actions/deliver/) and [supply](https://docs.fastlane.tools/actions/supply/) can then be used to upload to respective stores.
1. Works with FrameIt  
Works with Fastlane's FrameIt [text and background](https://docs.fastlane.tools/actions/frameit/#text-and-background) feature to add text, etc... to a framed screenshot generated by `screenshots`.  
1. Works with any attached real devices  
Use any number of real devices to capture screenshots.

Additional automation features:  
1. _Screenshots_ runs in the cloud.  
For live demo of _Screenshots_ running with the internationalized [example](example) app on macOS, linux and windows in cloud, see [below](#sample-run-on-travis-and-appveyor)    
1. _Screenshots_ works with any CI/CD tool.  
For live demo of uploading images, generated by _Screenshots_, to both store consoles, see demo of Fledge at https://github.com/mmcc007/fledge#demo

# Installation
On macOS:
````bash
brew update && brew install imagemagick
pub global activate screenshots
````

On linux:
````bash
# Debian, Ubuntu or Linux Mint
sudo apt-get install imagemagick
# Fedora, CentOS or RHEL
sudo yum install ImageMagick
# OpenSUSE
sudo zypper install imagemagick

pub global activate screenshots
````
Note: on linux ImageMagick is often already installed.

On windows:
````bash
choco install imagemagick.app
pub global activate screenshots
````
Note: ImageMagick v7 or later is recommended on windows.


If `pub` is not found, add to PATH using:

On macOS/Linux:  
```
export PATH="<path to flutter installation directory>/bin/cache/dart-sdk/bin:$PATH"
```
On windows:
```
set PATH=<path to flutter installation directory>\flutter\bin\cache\dart-sdk\bin;%APPDATA%\Pub\Cache\bin;%PATH%
```

Note: if running on Windows or Linux, can only run android devices/emulators. To also run on macOS use a CI that supports macOS.

# Usage

````
screenshots
````
Or, if using a config file other than the default 'screenshots.yaml':
````
screenshots -c <path to config file>
````
Other options:
```
usage: screenshots [-h] [-c <config file>] [-m <normal|recording|comparison|archive>] [-f <flavor>] [-b <true|false>] [-v]

sample usage: screenshots

-c, --config=<screenshots.yaml>                     Path to config file.
                                                    (defaults to "screenshots.yaml")

-m, --mode=<normal|recording|comparison|archive>    If mode is recording, screenshots will be saved for later comparison.
                                                    If mode is comparison, screenshots will be compared with recorded.
                                                    If mode is archive, screenshots will be archived (and cannot be uploaded via fastlane).
                                                    [normal (default), recording, comparison, archive]

-f, --flavor=<flavor name>                          Flavor name.
-b, --build=<true|false>                            Force build and install of app for all devices.
                                                    Override settings in screenshots.yaml (if any).
                                                    [true, false]

-v, --verbose                                       Noisy logging, including all shell commands executed.
-h, --help                                          Display this help information.
```

# Modifying tests for _Screenshots_
A special function is provided in the _Screenshots_ package that is called by the test to capture a screenshot. 
_Screenshots_ will then process the images appropriately during a _Screenshots_ run.

To capture screenshots in tests:
1. Add _Screenshots_ package in app's pubspec.yaml's dev_dependencies section  
   ````yaml
   dev_dependencies:
     screenshots: ^<current version>
   ````
2. In tests
    1. Import the dependency  
       ````dart
       import 'package:screenshots/screenshots.dart';
       ````
    2. Create the config at start of test  
       ````dart
            final config = Config();
       ````  
    3. Throughout the test make calls to capture screenshots  
       ````dart
           await screenshot(driver, config, 'myscreenshot1');
       ````
       
Note: make sure screenshot names are unique across all tests.

Note: to turn off the debug banner, in the integration test's main(), call:
````dart
  WidgetsApp.debugAllowBannerOverride = false; // remove debug banner for screenshots
````

## Modifying tests based on screenshots environment
In some cases it is useful to know what device, device type, screen size, screen orientation and locale the test is currently running with. To obtain this information use:
```
final screenshotsEnv = config.screenshotsEnv;
```
See https://github.com/flutter/flutter/issues/31609 for related `flutter driver` issue.

# Configuration
_Screenshots_ uses a configuration file to configure a run.  
 The default config filename is `screenshots.yaml`:
````yaml
# A list of screen capture tests
tests:
# Note: flutter driver expects a pair of files eg, main1.dart and main1_test.dart
  - test_driver/main1.dart 
  - test_driver/main2.dart 

# Interim location of screenshots from tests
staging: /tmp/screenshots

# A list of locales supported by the app
locales:
  - en-US
  - de-DE

# A map of devices to emulate
devices:
  ios:
    iPhone XS Max:
      frame: false
    iPad Pro (12.9-inch) (3rd generation):
      orientation: LandscapeRight
  android:
    Nexus 6P:

# Frame screenshots
frame: true
````

## Device Parameters
Individual devices can be configured in `screenshots.yaml` by specifying per device parameters.

| Parameter | Values | Required | Description |
| --- | --- | --- | --- |
|frame|true/false|optional|Controls whether screenshots generated on the device should be placed in a frame. Overrides the global frame setting (see example `screenshots.yaml` above).|
|orientation|Portrait \| LandscapeRight \| PortraitUpsideDown \| LandscapeLeft|optional|Controls orientation of device during test. Disables framing resulting in a raw screenshot. Ignored for real devices.|
|build|true(default)/false|optional|Builds and installs app. When set to false, can be used for pre-installed apps that need to be configured before running tests.|

_frame_ parameter notes:
- images generated for devices where framing is disabled may not be suitable for upload.
- set to true for devices unknown to _screenshots_.

_orientation_ parameter notes:
- multiple orientations can be specified. For example:
    ```yaml
        iPhone XS Max:
          orientation:
            - Portrait
            - LandscapeRight
    ```
- landscape orientation disables framing  
  This is because status/navigation bars in landscape mode are currently not implemented.
- orientation on iOS simulators is implemented using an AppleScript script which requires granting permission on first use.

## Test Options
In addition to using the default flutter driver mode, tests can also be specified using flutter driver parameters. For example:
```
tests:
  - --target=test_driver/main1.dart --driver=test_driver/main1_test1.dart
  - --target=test_driver/main2.dart --driver=test_driver/main2_test1.dart
``` 

# Record/Compare Mode
_Screenshots_ can be used to monitor any unexpected changes to the UI by comparing the new screenshots to previously recorded screenshots. Any differences will be highlighted in a 'diff' image for review.  

To use this feature:
1. Add a recording directory to `screenshots.yaml`
    ```yaml
    recording: /tmp/screenshots_record
    ```
1. Run a recording to capture expected screenshots:
    ```
    screenshots -m recording
    ```
1. Run subsequent _Screenshots_ with:
    ```
    screenshots -m comparison
    ```
    _Screenshots_ will compare the new screenshots with the recorded screenshots and generate a 'diff' image for each screenshot that does not compare. The diff image highlights the differences in red.

# Archive Mode
To generate screenshots for local use, such as generating reports of changes to UI over time, etc... use 'archive' mode. 

To enable this mode:
1. Add an archive directory to screenshots.yaml:
    ```yaml
    archive: /tmp/screenshots_archive
    ```
1. Run _Screenshots_ with:
    ````
    $ screenshots -m archive
    ````

# Integration with Fastlane
Since _Screenshots_ is intended to be used with Fastlane, after _Screenshots_ completes, the images can be found in the project at:
````
android/fastlane/metadata/android
ios/fastlane/screenshots
````
Images are in a format suitable for upload via [deliver](https://docs.fastlane.tools/actions/deliver/) 
and [supply](https://docs.fastlane.tools/actions/supply/).

Tip: One way to use _Screenshots_ with Fastlane is to call _Screenshots_ before calling Fastlane (or optionally call from Fastlane). Fastlane (for either iOS or Android) will then find the images in the appropriate place.  

(For a live demo of using Fastlane to upload screenshot images to both store consoles, see demo of Fledge at https://github.com/mmcc007/fledge#demo)

## Fastlane FrameIt
iOS images generated by _Screenshots_ can also be further processed using FrameIt's [text and background](https://docs.fastlane.tools/actions/frameit/#text-and-background) feature.

# Adding a device

1. Confirm device is present in [screens.yaml](https://github.com/mmcc007/screenshots/blob/master/lib/resources/screens.yaml).  
2. Add device to the list of devices in screenshots.yaml.  
3. Confirm a real device is attached, or install an emulator/simulator for device.   

If screen is not available, disable framing by setting `frame: false` for the device.

## Config validation
_Screenshots_ will check configuration before running for errors and provide a guide on how to resolve.

# Adding a screen

If device does not have a screen in screens.yaml please create an issue to request a new screen.

To submit a new screen please see related [README](https://github.com/mmcc007/screenshots/blob/master/test/resources/README.md).

# Upgrading
To upgrade, simply re-issue the install command
````bash
$ pub global activate screenshots
````
Note: the _Screenshots_ version should be the same for both the command line and in the `pubspec.yaml`.   
1. If upgrading the command line version of _Screenshots_, also upgrade
 the version of _Screenshots_ in the pubspec.yaml.    
2. If upgrading the version of _Screenshots_ in pubspec.yaml, also upgrade the command line version.

To check the version of _Screenshots_ currently installed:
```
pub global list
```

# Sample run on Travis and Appveyor
To view _Screenshots_ running with the internationalized [example](example) app on macOS and linux in the cloud see:  
https://travis-ci.com/mmcc007/screenshots

To view the images generated by _Screenshots_ during a run on travis see:  
https://github.com/mmcc007/screenshots/releases/

To view a similar run on windows see:  
https://ci.appveyor.com/project/mmcc007/screenshots

* Running _Screenshots_ in the cloud is  useful for automating the generation of screenshots in a CI/CD environment.  
* Running _Screenshots_ on macOS in the cloud can be used to generate screenshots when developing on Linux and/or Windows.

# Related projects
To run integration tests on real devices in cloud:  
https://github.com/mmcc007/sylph

To automate releases to both stores:  
https://github.com/mmcc007/fledge

# Issues and Pull Requests
[Issues](https://github.com/mmcc007/screenshots/issues) and 
[pull requests](https://github.com/mmcc007/screenshots/pulls) are welcome.