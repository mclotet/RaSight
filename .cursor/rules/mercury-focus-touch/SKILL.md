---
name: mercury-focus-touch
description: MercurySDK Focus & Temple Touch System.Applicable for TempleAction handling, focus object management, dynamic focus items, and fixed focus tracking.
---

## 5. Focus & Touch
### 5.1 IFocusable
**Package**: `com.ffalcon.mercury.android.sdk.focus`
A focus capability interface for custom focus objects (e.g., `focusObj` of FixPosFocusTracker, trackers of RecyclerView, etc.).

**Notes (Since / Threading / Lifecycle)**
- **Since**: Abstract interface for the unified focus system.
- **Threading**: It is recommended to read and write `hasFocus` in the main thread to ensure consistency with the UI state.
- **Lifecycle**: `focusParent` is recommended to point to a reachable object on the current page; disconnect cross-page focus reference chains when the page is destroyed.

| Property       | Type            | Description                                                              |
|----------------|-----------------|--------------------------------------------------------------------------|
| `hasFocus`     | `Boolean`       | Whether the current object has focus.                                    |
| `focusParent`  | `IFocusable?`   | Parent focus object; the focus can be returned to the parent object when releasing focus. |

---
### 5.2 reqFocus / releaseFocus (Extension Functions)
**Package**: `com.ffalcon.mercury.android.sdk.focus`
```kotlin
fun IFocusable.reqFocus(parent: IFocusable? = null)
fun IFocusable.releaseFocus()
```
- `reqFocus(parent)`: Sets the current object as focused; if `parent` is passed, `focusParent` will be set.
- `releaseFocus()`: Cancels the current focus and calls `focusParent?.reqFocus()` to return the focus to the parent object.

---
### 5.3 FocusHolder
**Package**: `com.ffalcon.mercury.android.sdk.ui.util`
A fixed-order focus list manager: maintains a set of `FocusInfo`, supports `next()` / `previous()` for focus switching, and allows looping.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Core management class for focus list switching.
- **Threading**: Internal state is not thread-safe; it is recommended to call only in the main thread.
- **Lifecycle**: List items are usually consistent with the page View lifecycle; it is recommended to synchronously verify the validity of the current focus item after dynamically deleting items.

| Constructor Parameter        | Description                                   |
|-----------------------------|-----------------------------------------------|
| `loop: Boolean = false`     | Whether to switch in a loop (wrap around after reaching the start/end). |

**Methods**
| Method                                                | Description                                                                 |
|-------------------------------------------------------|-----------------------------------------------------------------------------|
| `addFocusTarget(vararg focusInfoList: FocusInfo)`     | Adds items that can participate in focus switching.                         |
| `removeFocusTarget(target: Any)`                      | Removes the focus item associated with `target`; switches to the next item if the current focus is exactly this item. |
| `currentFocus(target: Any)`                           | Sets the current focus to the item corresponding to the specified `target` (commonly used to set the default focus). |
| `next()`                                              | Switches to the next item.                                                   |
| `previous()`                                          | Switches to the previous item.                                               |

**Properties**
| Property           | Type          | Description                  |
|--------------------|---------------|------------------------------|
| `currentFocusItem` | `FocusInfo`   | The item with current focus. |

---
### 5.4 FocusInfo
**Package**: `com.ffalcon.mercury.android.sdk.ui.util`
Represents a focus item that can be managed by FocusHolder.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Basic unit of the focus model.
- **Threading**: `eventHandler` and `focusChangeHandler` are triggered by the gesture distribution thread by default; it is recommended to only perform main thread UI operations in actual use.
- **Lifecycle**: `target` is often a View; ensure it is still attached to a valid page before executing focus changes.

**Construction**
```kotlin
FocusInfo(
    target: Any,                              // Arbitrary object for identification (e.g., View)
    eventHandler: (TempleAction) -> Unit,       // Temple gesture callback
    focusChangeHandler: (hasFocus: Boolean) -> Unit  // Callback when focus is gained/lost
)
```
| Method               | Description                                 |
|----------------------|---------------------------------------------|
| `fetchFocus()`       | Internally calls `focusChangeHandler(true)`, indicating focus gain. |
| `releaseFocus()`     | Internally calls `focusChangeHandler(false)`, indicating focus loss. |

**Example**
```kotlin
FocusInfo(
    btnAdd,
    eventHandler = { action ->
        when (action) {
            is TempleAction.Click -> addDynamicFocus()
            else -> Unit
        }
    },
    focusChangeHandler = { hasFocus ->
        mBindingPair.updateView {
            triggerFocus(hasFocus, btnAdd, mBindingPair.checkIsLeft(this))
        }
    }
)
```
---
### 5.5 FixPosFocusTracker
**Package**: `com.ffalcon.mercury.android.sdk.ui.util`
Fixed position focus switching tracker: Switches focus in the FocusHolder's list according to temple sliding (or single-step up/down/left/right), and can pass unhandled gestures to the `eventHandler` of the current focus item.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Used for gesture-driven switching of fixed focus items (or fixed areas).
- **Threading**: `handleFocusTargetEvent()` needs to be in the same thread as UI focus updates (main thread recommended).
- **Lifecycle**: Responds to events only when `focusObj.hasFocus == true`; switch this state in a timely manner when the page is switched or a pop-up seizes the focus.

**Construction**
```kotlin
FixPosFocusTracker(
    focusHolder: FocusHolder,
    continuous: Boolean = false,   // Whether to switch according to continuous sliding distance
    isVertical: Boolean = true,    // true for vertical sliding switch, false for horizontal
    ignoreDelta: Int = IGNORE_DELTA
)
```
| Member                                             | Type                              | Description                                                                 |
|----------------------------------------------------|-----------------------------------|-----------------------------------------------------------------------------|
| `focusHolder`                                      | `FocusHolder`                     | Managed focus list.                                                         |
| `focusObj`                                         | `IFocusable`                      | Used to indicate "whether the current tracker has focus", can be bound to the interface (e.g., for RecyclerView or the entire page). |
| `onFocusChangeListener`                            | `OnTrackerFocusChangeListener?`   | Focus change callback.                                                      |
| `handleFocusTargetEvent(action: TempleAction)`     | Method                            | Executes `next`/`previous` or forwards to the current `FocusInfo.eventHandler` according to `TempleAction` when focus is gained. |

**Constant**: `IGNORE_DELTA = 50`, the switch is triggered only when the distance exceeds this value in continuous sliding mode.

**Example**
```kotlin
val focusHolder = FocusHolder(true)
focusHolder.addFocusTarget(focusInfo1, focusInfo2)
focusHolder.currentFocus(btnAdd)
fixPosFocusTracker = FixPosFocusTracker(focusHolder).apply {
    focusObj.hasFocus = true
}
// In the collect of templeActionViewModel.state:
fixPosFocusTracker?.handleFocusTargetEvent(it)
```
---
### 5.6 addFocusView (BindingPair Extension)
**Package**: `com.ffalcon.mercury.android.sdk.ui.util`
Dynamically adds a pair of left and right focus Views to an existing mirrored layout, adds them to the specified `FocusHolder`, and automatically creates `FocusInfo`, synchronizes left and right views and 3D effects.
```kotlin
fun <T : ViewBinding, V : View> BindingPair<T>.addFocusView(
    parent: ViewGroup,
    viewFactory: () -> V,
    focusHolder: FocusHolder,
    focusConfig: FocusConfig<V>.() -> Unit = {}
): FocusViewHandle<V>
```
| Parameter      | Description                                                                 |
|---------------|-----------------------------------------------------------------------------|
| `parent`      | Parent container in the left layout (only pass the left one); the corresponding parent container on the right will be automatically found through Binding mapping. |
| `viewFactory` | Factory for creating a single View (called twice to generate left and right respectively). |
| `focusHolder` | The FocusHolder to add to.                                                  |
| `focusConfig` | Configuration for `FocusConfig<V>`.                                         |

**FocusConfig&lt;V&gt;** Optional Configuration:
| Property                | Type                                  | Description                                                              |
|-------------------------|---------------------------------------|--------------------------------------------------------------------------|
| `eventHandler`          | `((TempleAction) -> Unit)?`           | Gesture processing for this focus item.                                  |
| `onFocusChange`         | `((V, Boolean, Boolean) -> Unit)?`    | (view, hasFocus, isLeft) Update appearance when focus changes.           |
| `autoRequestFocus`      | `Boolean`                             | Whether to request focus immediately after addition.                     |
| `layoutParamsFactory`   | `((ViewGroup, V) -> LayoutParams)?`   | Custom LayoutParams for left and right sub Views.                        |

**Return Value**: `FocusViewHandle<V>`, which can be used to remove or request focus later.

---
### 5.7 FocusViewHandle<V : View>
**Package**: `com.ffalcon.mercury.android.sdk.ui.util`
Returned by `addFocusView`, used to manage dynamically added focus Views.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Matching handle for dynamic focus item capabilities.
- **Threading**: `clearFocusView()/updateView()` involve View tree modifications and should be executed in the main thread.
- **Lifecycle**: It is recommended to clean up unused dynamic focus items before the page is destroyed to prevent residual invalid references.

| Method                                | Description                                                                 |
|---------------------------------------|-----------------------------------------------------------------------------|
| `clearFocusView()`                    | Removes the left and right Views from the parent container and removes the focus item from the FocusHolder. |
| `requestFocus()`                      | Makes the FocusHolder set the current focus to this View.                   |
| `updateView(block: V.() -> Unit)`     | Executes `block` on both left and right Views simultaneously for synchronous update. |

---
### 5.8 TempleAction (Sealed Class)
**Package**: `com.ffalcon.mercury.android.sdk.touch`
Represents various gestures generated by temple touch control, received in `templeActionViewModel.state` or callbacks of Dialog/RecyclerView.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Unified semantic model for gesture streams of BaseEventActivity.
- **Threading**: Actions are data objects and can be passed across threads; however, the main thread must be used for UI updates after consumption.
- **Lifecycle**: It is recommended to collect in combination with `repeatOnLifecycle` to avoid continuing to consume gesture events when the page is invisible.

| Subtype                                           | Description                     |
|---------------------------------------------------|---------------------------------|
| `Idle`                                            | No operation                    |
| `Click`                                           | Single click                    |
| `LongClick`                                       | Long click                      |
| `DoubleClick`                                     | Double click                    |
| `TripleClick`                                     | Triple click                    |
| `SlideBackward(args)`                             | Slide backward (e.g., right slide) |
| `SlideForward(args)`                              | Slide forward (e.g., left slide) |
| `SlideUpwards(args)`                              | Slide upwards                   |
| `SlideDownwards(args)`                            | Slide downwards                 |
| `SlideContinuous(delta, longClick, vertical)`     | Continuous slide, `delta` is the difference from the press point |
| `MoveUp(isLongClick)`                             | Finger lift                     |
| `ActionUp`                                        | End of event sequence starting from DOWN |
| `ActionDown`                                      | Press down                      |
| `DoubleFingerClick` / `DoubleFingerLongClick`     | Double-finger click / long click |

All Actions in the form of data classes have `consumed: Boolean`, which can be marked as consumed in the business to avoid repeated processing.

**Slide Direction Semantic Description (Supplemented by X3 Document)**
The actual trigger directions of `SlideForward` and `SlideBackward` are affected by the system's "natural mode/non-natural mode" settings.
It is recommended that business logic does not hardcode these two as fixed "left/right slides", but designs interactions according to the "forward/backward" semantics, or provides user-configurable mapping in the settings page.

### 5.9 CommonTouchCallback (Compatibility Note)
**Package**: `com.ffalcon.mercury.android.sdk.touch`
Although the App does not directly implement this class, it is the underlying callback interface for event distribution of `BaseTouchActivity`. According to the X3 document and source code, the following capabilities can be noted:

**Notes (Since / Threading / Lifecycle)**
- **Since**: X3 supplements vertical sliding, double-finger and axis filtering capabilities.
- **Threading**: Callbacks are usually in the input event processing chain; it is recommended to keep them lightweight and return as soon as possible.
- **Lifecycle**: If the callback object is held by a custom component by itself, the association must be released when the component is destroyed to avoid continuing to receive events.

- New vertical sliding: `onTPSlideUpwards`, `onTPSlideDownwards`
- New double-finger events: `onTPDoubleFingerClick`, `onTPDoubleFingerLongClick`
- `onTPSlideContinuous(delta, longClick, vertical)` adds the `vertical` parameter
- `filterMode` (`NoFilter` / `OnlyX` / `OnlyY`) takes effect only on X3 and can filter continuous sliding axis data

> For the app calling surface covered in this document, these capabilities will eventually be mapped to the corresponding subtypes of `TempleAction` and uniformly consumed by `TempleActionViewModel.state`.

---
### 5.10 TempleActionViewModel
**Package**: `com.ffalcon.mercury.android.sdk.touch`
ViewModel for receiving and distributing temple gestures in Activity.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Cooperates with `BaseEventActivity` to provide gesture event stream distribution.
- **Threading**: Uses `viewModelScope` and Flow internally; the subscription side switches threads as needed, and the main thread is recommended for UI subscriptions.
- **Lifecycle**: The ViewModel lifecycle follows the host component; it is recommended to collect `state` in the `RESUMED` or `STARTED` state.

| Member             | Type                         | Description                                                         |
|--------------------|------------------------------|---------------------------------------------------------------------|
| `userTempleAction` | `Channel<TempleAction>`      | Channel for sending gestures (generally sent internally by BaseEventActivity). |
| `state`            | `SharedFlow<TempleAction>`   | Subscribe to this Flow to handle gestures.                          |

**Example**
```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.RESUMED) {
        templeActionViewModel.state.collect { action ->
            when (action) {
                is TempleAction.DoubleClick -> finish()
                is TempleAction.Click -> showDialog()
                else -> Unit
            }
        }
    }
}
```
---
### 5.11 actionName (MotionEvent Extension)
**Package**: `com.ffalcon.mercury.android.sdk.ui.activity` (in the same file as `BaseTouchActivity`)
```kotlin
fun MotionEvent.actionName(): String
```
Converts `MotionEvent.action` into a human-readable string (e.g., `"ACTION_DOWN"`, `"ACTION_UP"`), convenient for logging or debugging.
---
