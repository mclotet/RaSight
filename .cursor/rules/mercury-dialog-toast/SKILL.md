---
name: mercury-dialog-toast
description: MercurySDK Dialog & Toast Components.Applicable for FDialog/FToast construction, pop-up focus switching and gesture event integration.
---

## 6. Dialog & Toast
### 6.1 FToast
**Package**: `com.ffalcon.mercury.android.sdk.ui.toast`
Toast with support for binocular mirroring and 3D effects.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Universal binocular mirrored prompt component of the SDK.
- **Threading**: `show()` is recommended to be called in the main thread.
- **Lifecycle**: Depends on `MercurySDK.mApplication`; `MercurySDK.init()` must be completed first; Toast is a short-lived transient UI and should not carry critical business processes.

| Method                                                                      | Description                          |
|-----------------------------------------------------------------------------|--------------------------------------|
| `show(msg: String, short: Boolean = true, yOffset: Int = 200.dp)`           | Displays text Toast.                 |
| `show(msgResId: Int, short: Boolean = true, yOffset: Int = 200.dp)`         | Uses string resource ID.             |
| `showCustom(msg, short, yOffset, bindingClz, initViewBlock)`                | Uses custom ViewBinding layout and 3D initialization. |

**Example**
```kotlin
FToast.show("Click Confirm")
FToast.show(R.string.message, short = false, yOffset = 300.dp)
```
---
### 6.2 FDialog
**Package**: `com.ffalcon.mercury.android.sdk.ui.dialog`
Dialog with support for binocular mirroring, temple touch control and focus switching.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Universal binocular mirrored pop-up component of the SDK.
- **Threading**: Builder configuration, `show()/dismiss()` and view updates must all be executed in the main thread.
- **Lifecycle**: `dismiss()` cancels the internal coroutine scope; it is recommended to ensure the pop-up is closed and related resources are released before the host's `onDestroy`.

**Construction / Usage**: Configured in a chain via `FDialog.Builder<T : ViewBinding>`.

| Builder Method                                                              | Description                                                                 |
|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| `setContentView(bindingClz, initViewBlock, params)`                         | Sets the content layout (ViewBinding class), initialization block and optional LayoutParams. |
| `setFocusTracker(focusTracker: FocusTracker)`                               | Sets the focus switching logic inside the dialog.                            |
| `setCancelable(cancelable: Boolean)`                                        | Whether the dialog can be canceled via the back key.                        |
| `setCanceledOnTouchOutside(cancel: Boolean)`                                | Whether the dialog can be canceled by clicking outside.                     |
| `setOnDismissListener(onDismiss)`                                           | Dismiss callback.                                                           |
| `setOnShowListener(onShow)`                                                 | Show callback.                                                               |
| `setEventHandler(handler: (TempleAction, DialogInterface) -> Unit)`         | Global gesture processing (e.g., double click to close); button-level events are processed in the TrackInfo of FocusTracker. |
| `build(): FDialog`                                                          | Builds the FDialog.                                                          |

**Builder Properties** (available after `setContentView`):
- `mPair: BindingPair<T>`: Left and right Bindings of the dialog content.
- `mFocusTracker: FocusTracker?`: The set focus tracker.

**Example**
```kotlin
FDialog.Builder<DialogTestBinding>(this)
    .setCancelable(true)
    .setCanceledOnTouchOutside(true)
    .setContentView(DialogTestBinding::class.java) { pair, dialog ->
        // Initialize pair.left / pair.right
    }
    .apply {
        val tracker = FocusTracker(true).apply {
            addFocusTarget(btnOkTrackInfo, btnCancelTrackInfo)
        }
        tracker.currentFocus(pair.left.btnCancel)
        setFocusTracker(tracker)
    }
    .setEventHandler { action, dialog ->
        if (action is TempleAction.DoubleClick) dialog.dismiss()
    }
    .build()
    .show()
```
---
### 6.3 FocusTracker (for Dialog)
**Package**: `com.ffalcon.mercury.android.sdk.ui.dialog`
Similar to `FocusHolder`, but used inside FDialog: manages a list of `TrackInfo`, supports `next()` / `previous()` and looping.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Specialized management class for the Dialog focus system.
- **Threading**: Focus switching and UI feedback should be executed in the main thread.
- **Lifecycle**: Should be used with the same lifecycle as a single Dialog instance; reusing the same instance across Dialogs is not recommended.

| Construction                | Description                  |
|-----------------------------|------------------------------|
| `FocusTracker(loop: Boolean)`| Focus loops when `loop == true`. |

| Method                                                | Description                  |
|-------------------------------------------------------|------------------------------|
| `addFocusTarget(vararg trackInfoList: TrackInfo)`     | Adds items for focus switching. |
| `currentFocus(target: Any)`                           | Sets the current focus item. |
| `next()` / `previous()`                               | Switches focus.              |

---
### 6.4 TrackInfo
**Package**: `com.ffalcon.mercury.android.sdk.ui.dialog`
Configuration of a single focusable item inside FDialog.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Focus item model for Dialog.
- **Threading**: Ensure the main thread is used for UI or dismiss operations in `eventHandler`.
- **Lifecycle**: `target` is recommended to be bound to the current content view of the Dialog; this configuration should be considered invalid after the Dialog is destroyed.

**Construction**
```kotlin
TrackInfo(
    target: Any,
    eventHandler: (action: TempleAction, dialog: DialogInterface) -> Unit,
    focusChangeHandler: (hasFocus: Boolean) -> Unit,
    isSelected: Boolean = false
)
```
- `eventHandler`: Processing when the control receives temple gestures (e.g., Click to confirm/cancel and dismiss).
- `focusChangeHandler`: Update UI when focus is gained/lost (need to use `pair.updateView` + `make3DEffectForSide` internally to synchronize left and right).
---