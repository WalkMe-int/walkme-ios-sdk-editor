![WalkMe](https://console.mobile.walkme.com/images/walkme.png)

# WalkMe Editor iOS SDK

The WalkMe Editor (power mode) SDK embeds the in-app editor so you can author and preview WalkMe Mobile content inside your app—without shipping new binaries for every content change.

## Requirements

- iOS **15.0** or later  
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
| `setTenantId(_:)` | Sets the tenant identifier (up to 50 characters). The value is persisted and attached to every outgoing event. Calling it again overwrites the previously stored value; pass `nil` to clear it. |
| `setVariable(key:value:)` | Sets a variable for targeting and segmentation. Kept in memory for the app process only; set again on each app launch. |
| `setLanguage(_:)` | Changes the active language for WalkMe content. |
| `sendEvent(name:attributes:)` | Sends a custom tracked event: `name` identifies the event; `attributes` is an optional dictionary of key/value data for analytics and WalkMe engagement reporting. `name` must match an API tracked event in the console; unknown names are ignored. |
| `setEventUserVars(_:)` | Sets string key/value pairs attached to events. Use `WalkMeEventUserVarsKey` for supported keys. See `WalkMe/WalkMe/Common/Player/Model/WalkMeEventUserVarsKey.swift`. |
| `startItem(byID:deepLink:)` | Force-plays a WalkMe item by its numeric ID (can be retrieved from the console), dismissing any currently displayed item. `deepLink` is an optional URL string (custom scheme, e.g. `myapp://path`) — if provided and valid, the SDK triggers the app's `application(_:open:options:)` / `scene(_:openURLContexts:)` handler so the host app can navigate to the destination screen before the item appears. If the deeplink fails to open, the item plays immediately. If called before items are loaded (e.g. right after `start`), the item starts once they are; only the latest such call is kept. |
| `handleURL(_:)` | Handles a WalkMe permalink. Returns `true` if the URL is a WalkMe permalink, `false` if your app should handle it. See [Permalinks](#permalinks). |
| `dismissItem()` | Dismisses the currently displayed item, if any. |
| `setAnalyticsHandler(_:)` | Registers a callback invoked for every analytics event the SDK emits. Pass `nil` to remove. Prefer setting `analyticsHandler` on `WalkMeStartOptions` instead so the handler is active from the first event. |
| `setItemCallbacksDelegate(_:)` | Registers a delegate to receive item lifecycle callbacks. Can also be set via `WalkMeStartOptions.itemCallbacksDelegate` before calling `start`. |
| `setExternalBridgeDelegate(_:)` | Registers a `WMExternalBridgeInterface` bridge for external non-native UI frameworks (for example - Flutter). |

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
| `eventType` | `WMPublicEventType` | Enum: `play`, `click`, `close`, `engagedElement`, `changeLanguage`, `activity`, `pageChange`, `na`, `goalReached`. |
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

### Permalinks

WalkMe permalinks let a link (from an email, a push notification, a web page, etc.) invoke SDK actions through a custom URL scheme.

**URL format (v1.0):**

```text
com.walkme.api.{systemGuid}://1.0/{action}?{query}
```

| Action | Path | Query | Equivalent API |
|--------|------|-------|----------------|
| Restart SDK | `restart_sdk` | — | `restart()` |
| Start item | `start_item` | `item_id` (required) | `startItem(byID:)` |
| Start item + redirect | `start_item` | `item_id` (required), `redirect` (URL-encoded app deep link) | `startItem(byID:deepLink:)` |
| Send tracked event | `send_event` | `name` (required); all other query items are sent as attributes | `sendEvent(name:attributes:)` |
| Set variable | `set_variable` | `key`, `value` (both required) | `setVariable(key:value:)` |
| Set end user ID | `set_user_id` | `user_id` (required) | `setUserId(_:)` |

**Example:** `com.walkme.api.c22c935518874267b946f5ae49b21d20://1.0/start_item?item_id=898437&redirect=myapp%3A%2F%2Forders`

Permalinks for another system GUID, unsupported versions, unknown actions or missing required parameters are ignored (logged when logs are enabled).

**1. Register the scheme** in your app's `Info.plist`, using the same system GUID you pass to `start(options:)`:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>com.walkme.api.YOUR_SYSTEM_GUID</string>
        </array>
    </dict>
</array>
```

**2. Forward incoming URLs** to the SDK:

```swift
// UIApplicationDelegate
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey: Any] = [:]) -> Bool {
    if WalkMePowerMode.handleURL(url) { return true }
    // your own URL handling
    return false
}

// UISceneDelegate — also handle connectionOptions.urlContexts in scene(_:willConnectTo:options:) for cold launches
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    for context in URLContexts where !WalkMePowerMode.handleURL(context.url) {
        // your own URL handling
    }
}

// SwiftUI
.onOpenURL { url in
    if !WalkMePowerMode.handleURL(url) {
        // your own URL handling
    }
}
```

Permalinks received before `start(options:)` are performed once the SDK starts (a queued `restart_sdk` is skipped, since that start already is the restart), and are dropped on `stop()`. Call `start(options:)` early (e.g. in `application(_:didFinishLaunchingWithOptions:)`) so a permalink that launches the app is handled.

### Tagging SwiftUI views

Attach a stable, unique identifier to a SwiftUI view so WalkMe can identify it independently of its text, position, or localization:

```swift
Button("Checkout") { ... }
    .walkMeTag("checkout-button")
```

- The tag is included in the element's description when the element is captured.
- Empty strings are ignored — `walkMeTag("")` behaves as if the modifier were not applied.
- When tagged views are nested or overlap, the most specific (smallest-area) tag wins.
- A tag covers everything inside the tagged view: tapping a tagged button's icon or a tagged card's title captures the button or card. Tag the view users tap, not its icon or label.
- For a container whose children are separate targets (a grid, list or section), pass `scope: .selfOnly` — `.walkMeTag("devices-grid", scope: .selfOnly)` — so untagged elements inside it are not captured as the container. Tagged elements inside a container always keep their own tags.
- The modifier installs a fully transparent overlay that does not affect layout, touch handling, or accessibility.
- When a captured element has a tag, WalkMe finds it by the tag alone — independent of layout, text, or position changes. If no view with that tag is on screen, the element is treated as not found (there is no fallback to positional matching), so keep tags stable across app versions.
- Tags must be unique per screen. If the same tag appears more than once, a tap resolves to the instance actually tapped, while a match made without a tap resolves to the first instance in view-hierarchy order (the first one currently on screen, if the item's Check Visibility option is enabled).

## Support

- [WalkMe Help Center](https://support.walkme.com)  
- Email: [support@walkme.com](mailto:support@walkme.com)  

## License

Commercial software. Use is subject to your agreement with WalkMe.

### SwiftUI scrolling and automatic button actions

Use `WalkMeScrollView(tag:content:)`, `WalkMeList(tag:content:)`, or
`WalkMeForm(tag:content:)` (iOS 14+) to let WalkMe find off-screen targets.
`WalkMeTable` supports tables on iOS 16+. Give each viewport a unique stable tag,
then apply `.walkMeTag` to individual targets and recapture them in Power Mode.

```swift
WalkMeList(tag: "catalog") {
    ForEach(0..<100) { index in
        Text("Product \(index)").walkMeTag("product-\(index)")
    }
}
```

`Button("Continue", action: advance).walkMeTag("continue", action: advance)` exposes its
registered action to WalkMe automatic steps. It requires a mounted view hierarchy;
system alert actions are unsupported.

Paging respects the find timeout, user interaction, and an explicit
`autoScroll = 0`. Cancellation stops paging; failed searches restore the initial
position unless the user is scrolling. The Objective-C bridges
`WMSwiftUIScrollLocator` and `WMSwiftUIButtonLocator` require main-thread calls.
See [auto-scroll integration and bridge methods](docs/SwiftUI-Auto-Scroll.md)
and [automatic button actions](docs/SwiftUI-Auto-Button.md) for details.
