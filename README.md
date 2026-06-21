![WalkMe](https://console.mobile.walkme.com/images/walkme.png)

# WalkMe Editor iOS SDK

The WalkMe Editor (power mode) SDK embeds the in-app editor so you can author and preview WalkMe Mobile content inside your app—without shipping new binaries for every content change.

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
| `start(options:)` | Starts the SDK with a `WalkMeStartOptions` instance. Set your WalkMe **system GUID** via `WalkMeStartOptions(systemGuid:)` and configure language, environment, and other fields on that object before calling `start`. |
| `stop()` | Stops the SDK. |
| `restart()` | Restarts the SDK, re-initializing with the current configuration. Equivalent to stopping and starting with the same options. |
| `setUserId(_:)` | Sets the end-user identifier for storage, analytics, and targeting. |
| `setVariable(key:value:)` | Sets a variable for targeting and segmentation. |
| `setLanguage(_:)` | Changes the active language for WalkMe content. |
| `sendEvent(name:attributes:)` | Sends a custom tracked event: `name` identifies the event; `attributes` is an optional dictionary of key/value data for analytics and WalkMe engagement reporting. |
| `setEventUserVars(_:)` | Sets string key/value pairs attached to events. Use `WalkMeEventUserVarsKey` for supported keys. See `WalkMe/WalkMe/Common/Player/Model/WalkMeEventUserVarsKey.swift`. |
| `startItem(byID:deepLink:)` | Force-plays a WalkMe item by its numeric ID (can be retrieved from the console), dismissing any currently displayed item. `deepLink` is an optional URL string (custom scheme, e.g. `myapp://path`) — if provided and valid, the SDK triggers the app's `application(_:open:options:)` / `scene(_:openURLContexts:)` handler so the host app can navigate to the destination screen before the item appears. If the deeplink fails to open, the item plays immediately. |
| `dismissItem()` | Dismisses the currently displayed item, if any. |
| `setAnalyticsHandler(_:)` | Registers a callback invoked for every analytics event the SDK emits. Pass `nil` to remove. Prefer setting `analyticsHandler` on `WalkMeStartOptions` instead so the handler is active from the first event. |
| `setItemCallbacksDelegate(_:)` | Registers a delegate to receive item lifecycle callbacks. Can also be set via `WalkMeStartOptions.itemCallbacksDelegate` before calling `start`. |

### Swift example

```swift
import WalkMeEditor

let options = WalkMeStartOptions(systemGuid: "<your-guid>")
// Set other WalkMeStartOptions properties as required by your integration.

WalkMePowerMode.start(options: options)
```

### Start options

`WalkMeStartOptions` configures startup behavior. Create it with `WalkMeStartOptions(systemGuid:)` (required WalkMe system GUID), then set optional fields as needed:

| Property | Role |
|----------|------|
| `systemGuid` | WalkMe system GUID (set via initializer; immutable). |
| `userId` | Optional end-user identifier applied when the SDK starts. |
| `language` | Default language code. |
| `logsEnabled` | Verbose logging. |
| `dataCenter` | enum for which data center to use (e.g. production vs other areas). See `WalkMe/WalkMe/Common/Player/Public/WalkMeDataCenter.swift`. |
| `environment` | Environment name. |
| `analyticMode` | `WMAnalyticModeOFF` / `WMAnalyticModeON`. |
| `analyticsHandler` | Optional closure `((WMPublicAnalyticsDataInfo) -> Void)?` called for every analytics event. Setting it here is preferred over calling `setAnalyticsHandler(_:)` after start, so no early events are missed. |
| `itemCallbacksDelegate` | Optional delegate to receive item lifecycle callbacks (`itemWillShow` / `itemDidDismiss`). |

See `WalkMe/WalkMe/Common/Player/Public/WalkMeStartOptions.swift` for full definitions.

### Analytics callback

`WMPublicAnalyticsDataInfo` carries two fields:

| Field | Type | Description |
|-------|------|-------------|
| `eventType` | `WMPublicEventType` | Enum: `play`, `click`, `close`, `sessionStarted`, `engagedElement`, `changeLanguage`, `activity`, `pageChange`, `na`. |
| `payload` | `[String: Any]` | Full analytics payload for the event. |

```swift
import WalkMeEditor

let options = WalkMeStartOptions(systemGuid: "<your-guid>")
options.analyticsHandler = { info in
    print("WalkMe event:", info.eventType, info.payload)
}
WalkMePowerMode.start(options: options)
```

See `WalkMe/WalkMe/Common/Player/Public/WalkMeStartOptions.swift` for full definitions.

### Item callbacks

Implement `WMItemCallbacksDelegate` to be notified when a WalkMe item is about to appear or has been dismissed:

```swift
class MyDelegate: WMItemCallbacksDelegate {
    func itemWillShow(_ itemInfo: WalkMeItemInfo) {
        print("Item will show: \(itemInfo.itemId) (\(itemInfo.itemType))")
    }

    func itemDidDismiss(_ itemInfo: WalkMeItemInfo) {
        print("Item dismissed: \(itemInfo.itemId)")
    }
}
```

Register via `WalkMeStartOptions` (recommended — delegate is set before the SDK starts):

```swift
let options = WalkMeStartOptions(systemGuid: "<your-guid>")
options.itemCallbacksDelegate = myDelegate
WalkMePowerMode.start(options: options)
```

Or register at any time after start:

```swift
WalkMePowerMode.setItemCallbacksDelegate(myDelegate)
```

`WalkMeItemInfo` provides context about the item:

| Property | Description |
|----------|-------------|
| `itemId` | Numeric item ID (matches the ID in the WalkMe console). |
| `itemType` | Type string: `"Flow"`, `"ShoutOut"`, or `"Launcher"`. |
| `userData` | `WalkMeUserInfo` snapshot with device and app context at the time of the callback. |
| `action` | Dismiss reason string. `nil` for `itemWillShow` or when the reason is unknown. |

## Support

- [WalkMe Help Center](https://support.walkme.com)  
- Email: [support@walkme.com](mailto:support@walkme.com)  

## License

Commercial software. Use is subject to your agreement with WalkMe.
