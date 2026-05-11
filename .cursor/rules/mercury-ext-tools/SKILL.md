---
name: mercury-ext-tools
description: MercurySDK Extensions & Utility Functions.Applicable for dp/sp conversion, batch visibility control, unified logging, and implementation of debugging recommendations.
---

## 8. Extensions & Utilities
### 8.1 dp / sp (Extension Properties)
**Package**: `com.ffalcon.mercury.android.sdk.ext`
Converts values to density-independent px (based on `Resources.getSystem().displayMetrics`).
```kotlin
val Int.dp: Int
val Float.dp: Float
val Int.sp: Int
val Float.sp: Float
```
**Example**: `200.dp`, `41.dp`, `15f.dp`.

---
### 8.2 setViewVisible
**Package**: `com.ffalcon.mercury.android.sdk.ext`
```kotlin
fun setViewVisible(visible: Boolean, vararg views: View)
```
Unifiedly sets multiple Views to `View.VISIBLE` or `View.GONE`.

---
### 8.3 FLogger
**Package**: `com.ffalcon.mercury.android.sdk.util`
Unified logging utility with the TAG `"MercurySDK"`.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Built-in unified logging facade of the SDK.
- **Threading**: The interface can be called in any thread; long logs will be output in segments.
- **Lifecycle**: `isDebug` can be updated at runtime; it is recommended to call `updateLogSwitch()` to synchronize the state after debugging switch actions.

| Method                                     | Description                                                              |
|--------------------------------------------|--------------------------------------------------------------------------|
| `updateLogSwitch()`                        | Updates `isDebug` according to BuildConfig and `log.tag.MercurySDK`.     |
| `v(tag, msg)` / `v(msg)`                   | Verbose log level.                                                       |
| `d(tag, msg)` / `d(msg)`                   | Debug log level.                                                         |
| `i(tag, msg)` / `i(msg)`                   | Info log level.                                                          |
| `w(tag, msg)` / `w(msg)`                   | Warn log level.                                                          |
| `e(tag, msg, t?)` / `e(msg)` / `e(t?)`     | Error log level (supports throwable).                                    |
| `printStack()`                             | Prints the current call stack.                                           |

Log content is accompanied by the call location (class name, method name, file name, line number), convenient for jumping in Logcat.

---
### 8.4 Development & Debugging Suggestions (From RayNeo X3 ARSDK Document)
- It is recommended to set the theme `windowBackground` to pure black (`#FF000000`) for a more natural perspective effect.
- Release high-frequency capabilities such as sensors, Camera, and GPS in `onPause`/`onDestroy` in a timely manner to avoid background power consumption and resource occupation.
- For monocular screen projection debugging, refer to the `scrcpy --crop` solution in the SDK document.
---
