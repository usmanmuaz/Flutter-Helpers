# USMAN MUAZ
# Facebook SDK Setup (iOS — Flutter + CocoaPods)

A step-by-step guide to integrate **Facebook iOS SDK** into a **Flutter** app using **CocoaPods**, enable App Events (installs/opens), and verify everything end-to-end. Drop this file into your project or community repo as `FACEBOOK_SDK_SETUP.md`.

---

## TL;DR

1. Create a Facebook App in the Facebook Developer portal and note your **App ID**.
2. Add `FBSDKCoreKit`, `FBSDKLoginKit`, `FBSDKShareKit` to your iOS `Podfile` and run `pod install`.
3. Update `Info.plist` (FacebookAppID, URL scheme, LSApplicationQueriesSchemes, ATT string).
4. Update `AppDelegate.swift` to initialize the Facebook SDK and activate App Events.
5. (Flutter) Add `facebook_app_events` and log a test event to verify in Meta Events Manager.

---

## Prerequisites

* macOS with Xcode (recommended latest stable Xcode).
* CocoaPods installed (`sudo gem install cocoapods` or `brew install cocoapods`).
* Flutter installed and your project created (`flutter create my_app`).
* Your iOS bundle identifier (set in Xcode) ready.
* A Facebook App in Facebook Developers portal (you will need the App ID).

---

## Step 0 — Create & configure your Facebook App

1. Go to the Facebook Developer portal and create a new app.
2. Under **Settings → Basic** copy the **App ID** (you'll need it later).
3. Add the **iOS** platform for your app and enter your app's **Bundle ID**.
4. (Optional) Fill the App Store ID, and configure other settings (platform-specific).

---

## Step 1 — Add Facebook SDK pods to your Podfile

Open `ios/Podfile` and add the Facebook pods inside the `target 'Runner' do` block.

Example (typical Flutter Podfile section):

```ruby
platform :ios, '12.0'

target 'Runner' do
  use_frameworks!    # <-- uncomment if you run into module errors (see Troubleshooting)
  # use_modular_headers! # optional; try only if you hit header/module errors

  # Flutter generated pods (do not remove) ---
  use_frameworks!
  use_modular_headers!
  flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))
  # --- end Flutter pods

  # Facebook SDK
  pod 'FBSDKCoreKit'
  pod 'FBSDKLoginKit'
  pod 'FBSDKShareKit'
end
```

**Notes:**

* If your Flutter `Podfile` is different, just ensure the three `pod` lines are inside the `Runner` target.
* If you get `module 'FBSDKCoreKit' not found` or similar errors, see Troubleshooting (we cover `use_frameworks!` / `use_modular_headers!`).

---

## Step 2 — Install Pods

From the `ios/` folder run:

```bash
pod repo update
pod install
```

If you ever hit conflicts, try:

```bash
rm -rf Pods Podfile.lock
pod install --repo-update
```

Open the workspace after install:

```bash
open Runner.xcworkspace
```

> **Always** open the `.xcworkspace` file — not the `.xcodeproj` — once CocoaPods is used.

---

## Step 3 — Info.plist changes

Open `ios/Runner/Info.plist` and add the following keys (replace `YOUR_FACEBOOK_APP_ID` and `YOUR_APP_NAME`):

```xml
<key>FacebookAppID</key>
<string>YOUR_FACEBOOK_APP_ID</string>
<key>FacebookDisplayName</key>
<string>YOUR_APP_NAME</string>

<key>LSApplicationQueriesSchemes</key>
<array>
  <string>fbapi</string>
  <string>fb-messenger-api</string>
  <string>fbauth2</string>
  <string>fbshareextension</string>
</array>

<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>fbYOUR_FACEBOOK_APP_ID</string>
    </array>
  </dict>
</array>

<!-- App Tracking Transparency (if you plan to use tracking) -->
<key>NSUserTrackingUsageDescription</key>
<string>We use your advertising identifier to provide personalized ads and measurement.</string>
```

**Optional Facebook settings** (put in Info.plist if you want auto logging or explicit flags):

```xml
<key>FacebookAutoLogAppEventsEnabled</key>
<true/>
<key>FacebookAdvertiserIDCollectionEnabled</key>
<true/>
```

---

## Step 4 — Update `AppDelegate.swift` (Flutter + Swift)

Edit `ios/Runner/AppDelegate.swift` to initialize the Facebook SDK and App Events. Example full file (Flutter project with Swift AppDelegate):

```swift
import UIKit
import Flutter
import Firebase
import FBSDKCoreKit

@main
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    // Firebase
    FirebaseApp.configure()

    // Facebook SDK initialization
    ApplicationDelegate.shared.application(
      application,
      didFinishLaunchingWithOptions: launchOptions
    )

    // Optional: print SDK version to verify
    print("Facebook SDK version: \(Settings.shared.sdkVersion)")

    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  // Track activations (app opens)
  override func applicationDidBecomeActive(_ application: UIApplication) {
    AppEvents.shared.activateApp()
  }

  // Handle incoming URLs (login callbacks)
  override func application(
    _ app: UIApplication,
    open url: URL,
    options: [UIApplication.OpenURLOptionsKey : Any] = [:]
  ) -> Bool {
    if ApplicationDelegate.shared.application(app, open: url, options: options) {
      return true
    }
    return super.application(app, open: url, options: options)
  }
}
```

**Notes:**

* The `Settings.shared.sdkVersion` line prints the SDK version to the Xcode console (helpful for verification).
* `AppEvents.shared.activateApp()` ensures app opens / installs are recorded. Newer SDKs prefer instance-based APIs (`AppEvents.shared...` and `Settings.shared...`).

---

## Step 5 — Flutter: add `facebook_app_events` and log a test event

In your `pubspec.yaml`:

```yaml
dependencies:
  facebook_app_events: ^0.19.0
```

Run:

```bash
flutter pub get
```

Then add a small test button in your Flutter UI to send an event:

```dart
import 'package:facebook_app_events/facebook_app_events.dart';

final facebookAppEvents = FacebookAppEvents();

// Example usage (on button tap)
facebookAppEvents.logEvent(
  name: 'test_event',
  parameters: {'button': 'pressed', 'time': DateTime.now().toIso8601String()},
);
```

**Run the app on a real device** and press the button — you should see the event appear in the **Test Events** tab inside Meta Events Manager.

---

## Step 6 — Verify integration (quick checklist)

1. Build and run the app on a **real iOS device** (some SDK features need a device).
2. In Xcode console you should see the printed SDK version from `Settings.shared.sdkVersion`.
3. In Meta Business Events Manager → select your app → open **Test Events** and trigger `test_event` from your device: it should appear.
4. For installs/opens look at Events Manager’s dashboard (may take \~15–30 minutes for analytics view).

---

## Troubleshooting (common issues)

### `module 'FBSDKCoreKit' not found` or compile errors

* Try adding `use_frameworks!` and/or `use_modular_headers!` at the top of your `target 'Runner' do` block in `Podfile`. Then:

```bash
cd ios
rm -rf Pods Podfile.lock
pod install --repo-update
```

* If that breaks other pods, try toggling `use_frameworks!` or using `:modular_headers => true` only for Facebook pods.

### CocoaPods version / dependency conflicts

* Update CocoaPods and repo specs:

```bash
sudo gem install cocoapods
pod repo update
```

* If a pod version conflict shows up, you might need to pin pod versions (see below) or update other plugin versions that depend on FBSDK.

### Events not appearing in Events Manager

* Make sure the device has network connectivity.
* Confirm `AppEvents.shared.activateApp()` (for app opens) and that you sent an explicit event via `facebook_app_events`.
* Verify App ID in `Info.plist` matches your Facebook App ID.
* Check that App Tracking Transparency (ATT) exists and user consent flows are handled: missing ATT may affect some signals.

### Xcode workspace vs project

* **Always open** the `Runner.xcworkspace` (not `.xcodeproj`) after `pod install`.

---

## Pinning SDK versions (optional)

If you want to pin a specific major version for stability:

```ruby
pod 'FBSDKCoreKit', '~> 16.0'
pod 'FBSDKLoginKit', '~> 16.0'
pod 'FBSDKShareKit', '~> 16.0'
```

Adjust the version to whatever is stable for your project and run `pod install`.

---

## Swift Package Manager (SPM) alternative

If you prefer SPM (Xcode native):

* In Xcode: `File → Add Packages...` and add `https://github.com/facebook/facebook-ios-sdk`.
* Select the packages you need (Core, Login, Share).

Keep in mind: if you use Flutter + CocoaPods plugin ecosystem, CocoaPods is usually the path of least friction.

---

## App Store / Privacy notes

* If you use advertising identifiers or personalized advertising, you must request App Tracking Transparency permission and include `NSUserTrackingUsageDescription`.
* You may need to declare data use in App Store Connect (privacy details) — check Apple & Facebook guidance.
* For SKAdNetwork / AEM support, consult Facebook Ads documentation and add required entries in your project if you run ad campaigns.

---

## Useful links

* Facebook iOS SDK — Getting Started: [https://developers.facebook.com/docs/ios/getting-started](https://developers.facebook.com/docs/ios/getting-started)
* App Events for iOS: [https://developers.facebook.com/docs/app-events/getting-started-app-events-ios/](https://developers.facebook.com/docs/app-events/getting-started-app-events-ios/)
* Facebook iOS SDK GitHub: [https://github.com/facebook/facebook-ios-sdk](https://github.com/facebook/facebook-ios-sdk)
* `facebook_app_events` Flutter plugin on pub.dev: [https://pub.dev/packages/facebook\_app\_events](https://pub.dev/packages/facebook_app_events)

---

## License & Attribution

This guide references Facebook / Meta official documentation and the `facebook_app_events` Flutter plugin. Follow their respective licenses and changelogs when upgrading.

---

If you want, I can:

* Create a short `README.md` badge and summary for your GitHub repo.
* Produce a minimal sample `Podfile` that matches your exact Flutter Podfile.
* Produce a small `main.dart` test screen that calls `facebook_app_events` and logs a few events.
