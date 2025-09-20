# USMAN MUAZ
## Crash occurred when compiling unknown function in unoptimized JIT mode in unknown pass
* A change to iOS has caused a temporary break in Flutter's debug mode on physical devices.
* See https://github.com/flutter/flutter/issues/163984 for details.

* Workarounds:
* Use iOS 18.3 or lower
* Use a simulator
* Or run on device with --release / --profile

# Solution

* 1- In Xcode → select your scheme → change “Build configuration” to Release instead of Debug when running on device.

# Using VSCode
* 1- Create launch.json file in project .vscode folder
* 2- Add these detail & then do clean & Pub get

{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Flutter Profile on iPhone",
            "request": "launch",
            "type": "dart",
            "flutterMode": "profile"
        },
        {
            "name": "Flutter Release on iPhone",
            "request": "launch",
            "type": "dart",
            "flutterMode": "release"
        }
    ]
}
