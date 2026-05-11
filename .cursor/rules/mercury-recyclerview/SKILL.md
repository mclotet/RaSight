---
name: mercury-recyclerview
description: MercurySDK RecyclerView Tracking & Snapping Capabilities.Applicable for focus-following sliding, follow-touch scrolling, SnapHelper configuration, and binocular list synchronization.
---

## 7. RecyclerView Support
### 7.1 RecyclerViewFocusTracker
**Package**: `com.ffalcon.mercury.android.sdk.ui.util`
RecyclerView focus and scroll tracker based on **continuous sliding distance**: Switches the currently selected item and scrolls according to the `delta` of `SlideContinuous`, suitable for lists with "focus following sliding".

**Notes (Since / Threading / Lifecycle)**
- **Since**: Core tracker for moving focus item lists.
- **Threading**: All RecyclerView and Adapter operations must be performed in the main thread.
- **Lifecycle**: It is recommended to process `handleActionEvent()` while the page is visible; set `focusObj.hasFocus` to `false` when the page is invisible or out of focus.

**Construction**
```kotlin
RecyclerViewFocusTracker(
    mPair: ViewPair<RecyclerView>,
    ignoreDelta: Int = IGNORE_DELTA,
    loop: Boolean = false
)
```
| Member                  | Type                              | Description                                                               |
|-------------------------|-----------------------------------|---------------------------------------------------------------------------|
| `mPair`                 | `ViewPair<RecyclerView>`          | Two left and right RecyclerViews.                                         |
| `focusObj`              | `IFocusable`                      | Whether to have focus (e.g., the list only responds when the entire page gains focus). |
| `currentSelectPos`      | `Int`                             | Current selected item position (writable via `setCurrentSelectPos`).      |
| `onItemFocusListener`   | `OnItemFocusListener?`            | Callback for selected item changes.                                       |
| `onFocusChangeListener` | `OnTrackerFocusChangeListener?`   | Callback for focus gain/loss.                                             |
| `refreshListener`       | `PullToRefreshListener?`          | Triggers "pull down to refresh", etc. when sliding forward continuously at the first item. |

**Methods**
| Method                                                                   | Description                                                                 |
|--------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| `setCurrentSelectPos(index: Int)`                                        | Sets the current selected index.                                            |
| `handleActionEvent(it: TempleAction, block: (TempleAction) -> Unit)`     | Processes gestures when focused; switches items on continuous sliding, and forwards Click/DoubleClick, etc. to `block`. |
| `checkedSelectPos(): Int`                                                | Returns the current selected position when focused, otherwise -1.           |
| `checkPosSelected(pos: Int): Boolean`                                    | Determines whether a position is currently selected and has focus.          |
| `notifyDataSetChanged()`                                                 | Calls `notifyDataSetChanged()` for the adapters of both left and right RecyclerViews. |

**Example**
```kotlin
favoriteTracker = RecyclerViewFocusTracker(
    ViewPair(mBindingPair.left.recyclerView, mBindingPair.right.recyclerView),
    ignoreDelta = 70
)
favoriteTracker.focusObj.hasFocus = true
// In state.collect:
favoriteTracker.handleActionEvent(it) { action ->
    when (action) {
        is TempleAction.Click -> { /* Click the current item */ }
        is TempleAction.DoubleClick -> finish()
        else -> {}
    }
}
```
---
### 7.2 RecyclerViewSlidingTracker
**Package**: `com.ffalcon.mercury.android.sdk.ui.util`
RecyclerView tracker based on **slide snap**: Relies on `SnapHelper` to determine the currently selected item when scrolling stops, and can subscribe to original touch events to achieve "follow-touch" scrolling.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Used for fixed focus + follow-touch scrolling scenarios.
- **Threading**: Event conversion, `dispatchTouchEvent`, and Adapter refresh need to be performed in the main thread.
- **Lifecycle**: The original event stream should be automatically started and stopped with the page focus state after registration; ensure no more events are received and forwarded before leaving the page.

**Construction**
```kotlin
RecyclerViewSlidingTracker(mPair: ViewPair<RecyclerView>)
```
**Methods**
| Method                                                           | Description                                                                 |
|------------------------------------------------------------------|-----------------------------------------------------------------------------|
| `observeOriginMotionEventStream(dispatcher, eventTransform)`     | Registers to `MotionEventDispatcher`, converts temple events into RecyclerView touch events to achieve follow-touch scrolling. |
| `setCurrentSelectPos(index: Int)`                                | Sets the current selected index.                                            |
| `handleActionEvent(it, block)`                                   | Processes gestures; forwards Click/DoubleClick to `block`, and `SlideBackward` at the first item can trigger `refreshListener`. |
| `smoothScrollToPosition(smoothScroll)`                           | Scrolls to `currentSelectPos`.                                              |
| `notifyItemChanged(pos)` / `notifyDataSetChanged()`              | Refreshes the specified item or the entire list.                            |

**Note**: It is necessary to set a `SnapHelper` (e.g., `StartSnapHelper`) for the RecyclerView, and set it via `setTag(R.id.tag_snap_helper, snapHelper)` to ensure `findSelectedPosition` works correctly. `R.id.tag_snap_helper` in the SDK is defined in MercurySDK's `res/values/id.xml`.

**Example**
```kotlin
favoriteTracker = RecyclerViewSlidingTracker(
    ViewPair(mBindingPair.left.recyclerView, mBindingPair.right.recyclerView)
)
favoriteTracker.observeOriginMotionEventStream(motionEventDispatcher) { event ->
    MotionEvent.obtain(
        event.downTime, event.eventTime, event.action,
        320f, event.x, event.metaState
    )
}
recyclerView.apply {
    val snapHelper = StartSnapHelper(41.dp)
    snapHelper.attachToRecyclerView(this)
    setTag(com.ffalcon.mercury.android.sdk.R.id.tag_snap_helper, snapHelper)
}
```
---
### 7.3 StartSnapHelper
**Package**: `com.ffalcon.mercury.android.sdk.util`
Inherits from `LinearSnapHelper`, makes the RecyclerView snap to the first visible item in the **start direction** (or with an offset), suitable for lists with "fixed focus items".

**Notes (Since / Threading / Lifecycle)**
- **Since**: Used for list start snap scenarios on glasses.
- **Threading**: As a RecyclerView UI component, it is only used in the main thread.
- **Lifecycle**: It is recommended to create and destroy it synchronously with the RecyclerView; it can be reattached as needed after replacing the Adapter/LayoutManager.

**Construction**
```kotlin
StartSnapHelper(offset2Start: Int)
```
- `offset2Start`: Offset relative to the start edge (e.g., 41.dp), used to align the snap point to the center of the first item, etc.

**Usage**: `attachToRecyclerView(recyclerView)`, and set `setTag(R.id.tag_snap_helper, snapHelper)` when used with `RecyclerViewSlidingTracker`.

---
### 7.4 BaseBindingHolder<T : ViewBinding> / SimpleBindingAdapter<B : ViewBinding>
**Package**: `com.ffalcon.mercury.android.sdk.ext`
- **BaseBindingHolder**: A `RecyclerView.ViewHolder` that holds a ViewBinding, access the layout via `binding`.
- **SimpleBindingAdapter**: An Adapter based on generic ViewBinding, inflates the corresponding Binding via reflection in `onCreateViewHolder`, and returns `BaseBindingHolder<B>`.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Basic encapsulation of RecyclerView + ViewBinding.
- **Threading**: Adapter lifecycle callbacks run in the main thread; avoid heavy calculations in `onBindViewHolder`.
- **Lifecycle**: Reflective inflation depends on generic signatures; the generic structure should be kept correct during obfuscation/refactoring.

When list items need left and right synchronization, the 3D effect and selected state can be updated in the Adapter according to `BindingPair.checkIsLeft` or the tracker's `checkPosSelected`.
---
