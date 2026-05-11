---
name: mercury-core-api
description: MercurySDK Core APIs (MobileState, DeviceUtil).Applicable for connection state monitoring, device type branching, and basic capability integration.
---

## 3. Core APIs
### 3.1 MobileState
**Package**: `com.ffalcon.mercury.android.sdk.api`
Used to observe the connection state with the RayNeo AR mobile App.

**Notes (Since / Threading / Lifecycle)**
- **Since**: `MobileState` provides connection state observation capability in X3 feature integration.
- **Threading**: The `Flow` can be collected in any coroutine context; switch back to the main thread for UI updates (or handle in the main thread environment of `lifecycleScope`).
- **Lifecycle**: Internally registers `ContentObserver` in `onStart` and automatically unregisters it in `onCompletion`; avoid long-term hanging collection in global coroutines without lifecycle constraints.

| Member                            | Type              | Description                                                              |
|-----------------------------------|-------------------|--------------------------------------------------------------------------|
| `METHOD_MOBILE_CONNECT_STATE`     | `String`          | ContentProvider method name constant: `"mobileConnectState"`.            |
| `isMobileConnected()`             | `Flow<Boolean>`   | Returns a Flow of the current mobile connection state, pushed automatically when the connection changes. |

**Example: Update UI based on BLE connection state**
```kotlin
MobileState.isMobileConnected()
    .onEach { connected ->
        mBindingPair.updateView {
            tvBleStatus.text = if (connected) "connect" else "disconnect"
        }
    }
    .launchIn(lifecycleScope)
```
---
### 3.2 DeviceUtil
**Package**: `com.ffalcon.mercury.android.sdk.util`
Device model judgment utility.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Added in the X3 adaptation phase to distinguish X2/X3 branch logic.
- **Threading**: `isX3Device()` is a lightweight synchronous call and can be invoked in any thread.
- **Lifecycle**: No binding to component lifecycle; it is recommended to read and cache the result in the page initialization phase for UI branch processing.

| Method                      | Description                     |
|-----------------------------|---------------------------------|
| `isX3Device(): Boolean`     | Determines if the current device is RayNeo X3. |

**Example**
```kotlin
if (DeviceUtil.isX3Device()) {
    tvDevicesType.text = "RayNeo X3"
    // Display BLE status, etc. only for X3
} else {
    tvDevicesType.text = "RayNeo X2"
}
```
---
