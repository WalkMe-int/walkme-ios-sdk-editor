![WalkMe](https://console.mobile.walkme.com/images/walkme.png)

# WalkMe Editor iOS SDK

The WalkMe Editor (power mode) SDK embeds the in-app editor so you can author and preview WalkMe Mobile content inside your app—without shipping new binaries for every content change.

> **Beta:** This SDK is currently in beta.

## Requirements

- iOS **14.0** or later  
- **Swift 5.0** or later  
- **Xcode** with Swift Package Manager support  

## Installation (Swift Package Manager)

Add the package dependency in Xcode:

1. **File → Add Package Dependencies…**
2. Enter the repository URL: **https://github.com/WalkMe-int/walkme-ios-sdk-editor**
3. Choose the **WalkMeEditor** product and add it to your app target.

Releases are version-tagged on that repo; pick the version that matches your WalkMe Mobile account and integration notes.

> **Note:** The distributed binary requires **Lottie** to be linked separately in the host app. If Xcode reports missing Lottie symbols, add [Lottie for iOS](https://github.com/airbnb/lottie-ios) to your target per your WalkMe integration checklist.

## Public API (`WalkMePowerMode`)

The primary integration surface is `WalkMePowerMode` in `WalkMe/WalkMeEditor/WMPowerModeSDK.swift`.

| Method | Description |
|--------|-------------|
| `start(withGUID:options:)` | Starts the SDK with your WalkMe **GUID** and `WalkMeStartOptions`. Use a configured instance—typically from `WalkMeStartOptions.defaults()`—so language, environment, and other startup settings are applied at launch. |
| `stop()` | Stops the SDK. |
| `setUserId(_:)` | Sets the end-user identifier for storage, analytics, and targeting. |
| `setUserAttribute(key:value:)` | Sets a user attribute for targeting and segmentation. |
| `setLanguage(_:)` | Changes the active language for WalkMe content. |
| `sendEvent(name:attributes:)` | Sends a custom tracked event: `name` identifies the event; `attributes` is an optional dictionary of key/value data for analytics and WalkMe engagement reporting. |

All methods are exposed to **Objective-C** (`@objc`).

### Swift example

```swift
import WalkMeEditor

let options = WalkMeStartOptions.defaults()
// Set other WalkMeStartOptions properties as required by your integration.

WalkMePowerMode.start(withGUID: "<your-guid>", options: options)
```

### Start options

`WalkMeStartOptions` (Objective-C class, usable from Swift) configures startup behavior. Use `WalkMeStartOptions.defaults` and override properties as needed:

| Property | Role |
|----------|------|
| `userId` | Optional end-user identifier applied when the SDK starts. |
| `language` | Default language code |
| `walkMeDataCenter` | Data center |
| `environment` | Environment name |
| `logsEnabled` | Verbose logging |
| `analyticMode` | `WMAnalyticModeOFF` / `WMAnalyticModeON` |

See `WalkMe/WalkMe/Common/Player/Public/WalkMeStartOptions.h` for full definitions.

## Support

- [WalkMe Help Center](https://support.walkme.com)  
- Email: [support@walkme.com](mailto:support@walkme.com)  

## License

Commercial software. Use is subject to your agreement with WalkMe.
