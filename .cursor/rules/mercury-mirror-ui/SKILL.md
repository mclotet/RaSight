---
name: mercury-mirror-ui
description: MercurySDK Binocular & Mirrored UI Capabilities.Applicable for implementing BindingPair, 3D parallax, and mirrored UI structures for Activity/Fragment/Container.
---

## 4. Binocular Mirrored Display UI
### 4.1 BindingPair&lt;B : ViewBinding&gt;
**Package**: `com.ffalcon.mercury.android.sdk.core`
A pair of left and right ViewBindings based on ViewBinding, where operations on the left layout can be synchronously mapped to the right, applicable for binocular mirrored pages.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Serves as the basic capability for mirrored rendering, used across Activity/Fragment/View-level components.
- **Threading**: `update/updateView/setLeft` are UI operations and should be executed in the main thread.
- **Lifecycle**: `BindingPair` shares the same lifecycle with the left and right Bindings it holds; do not continue to reference internal Views after the page is destroyed.

| Member         | Type      | Description                                          |
|----------------|-----------|------------------------------------------------------|
| `left`         | `B`       | Left ViewBinding.                                    |
| `right`        | `B`       | Right ViewBinding.                                   |
| `PARALLAX`     | `Float`   | Default parallax value (3f) for 3D effects.          |

**Methods**
| Method                                                   | Description                                                                 |
|----------------------------------------------------------|-----------------------------------------------------------------------------|
| `update(block: T.() -> Unit)`                            | Executes `block` on both `left` and `right` for synchronous update of left and right views. |
| `updateView(block: T.() -> Unit)`                        | Same as `update`, semantically emphasizing "view update".                    |
| `setLeft(block: T.() -> Unit)`                           | Executes `block` only on the left (e.g., register left-side events only).    |
| `checkIsLeft(t: T): Boolean`                             | Determines if the current ViewBinding is the left one.                       |
| `enable3DEffect(vararg leftViews, enable, parallax)`     | Enables/disables 3D parallax in batches for a set of left Views; only pass **left** Views, otherwise a null pointer may occur. |

**Note**: Do not call `enable3DEffect` for the "right View" inside the block of `updateView` / `update`. The internal logic uses the left View as the key to find the right one, and passing the right View will return null.

---
### 4.2 make3DEffect (Top-Level Function)
**Package**: `com.ffalcon.mercury.android.sdk.core`
Sets or cancels 3D parallax (horizontal translation) for a pair of left and right Views.
```kotlin
@JvmOverloads
fun make3DEffect(
    leftView: View,
    rightView: View,
    enable: Boolean = true,
    parallax: Float = BindingPair.PARALLAX
)
```
| Parameter      | Description                                 |
|---------------|---------------------------------------------|
| `leftView`    | Corresponding View for the left eye.        |
| `rightView`   | Corresponding View for the right eye.       |
| `enable`      | `true` to apply parallax, `false` to reset to 0. |
| `parallax`    | Offset: left View moves right, right View moves left. |

---
### 4.3 make3DEffectForSide (Top-Level Function)
**Package**: `com.ffalcon.mercury.android.sdk.core`
Sets 3D parallax for a single View on one side (left or right), commonly used for single-side views such as list items and buttons.
```kotlin
@JvmOverloads
fun make3DEffectForSide(
    view: View,
    isLeft: Boolean,
    enable: Boolean = true,
    parallax: Float = BindingPair.PARALLAX
)
```
| Parameter     | Description                                          |
|--------------|------------------------------------------------------|
| `view`       | The View to set parallax for.                        |
| `isLeft`     | `true` indicates the View belongs to the left layout. |
| `enable`     | Whether to enable parallax.                          |
| `parallax`   | Offset value.                                        |

**Example: Focus highlight + 3D for list items or buttons**
```kotlin
pair.updateView {
    val isLeft = pair.checkIsLeft(this)
    triggerFocus(hasFocus, btnOk, isLeft)  // e.g., change background color
}
// Inside custom triggerFocus:
make3DEffectForSide(view, isLeft, hasFocus)
```
---
### 4.4 ViewPair&lt;T : View&gt;
**Package**: `com.ffalcon.mercury.android.sdk.core`
Encapsulation of a pair of left and right Views of the same type, inherited from `BaseMirrorAction<T>`, providing `update`, `setLeft`, `checkIsLeft`, etc. Commonly used for synchronous operations of two left and right RecyclerViews.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Applicable for general scenarios of paired operations on left and right Views.
- **Threading**: Calls involving View updates should be made in the main thread.
- **Lifecycle**: Holds strong references to Views; it is recommended to hold only during the page's active period and avoid caching in static singletons.

**Construction**
```kotlin
ViewPair(mBindingPair.left.recyclerView, mBindingPair.right.recyclerView)
```
---
### 4.5 BaseMirrorActivity&lt;B : ViewBinding&gt;
**Package**: `com.ffalcon.mercury.android.sdk.ui.activity`
An Activity base class with left and right mirrored layout: automatically generates two left and right Bindings according to the generic type `B` and places them in a horizontally equally divided root layout; it also inherits temple touch control and gesture distribution (see [BaseEventActivity / Touch](#5-focus--touch)).

**Notes (Since / Threading / Lifecycle)**
- **Since**: Standard base class for binocular mirrored Activities.
- **Threading**: Gesture collection and UI updates are executed in the main thread in accordance with Android component conventions.
- **Lifecycle**: It is recommended to perform UI initialization in `onCreate`, start collecting events in `onStart/onResume`, and release resources such as players/sensors in `onStop/onDestroy`.

| Member                      | Type                      | Description                                                                 |
|-----------------------------|---------------------------|-----------------------------------------------------------------------------|
| `mBindingPair`              | `BindingPair<B>`          | Pair of left and right ViewBindings, used for `updateView`, `enable3DEffect`, `checkIsLeft`, etc. |
| `templeActionViewModel`     | `TempleActionViewModel`   | ViewModel for temple gestures, collect gestures through the `state` Flow.    |

**Usage**
1. Inherit and specify the ViewBinding type, e.g., `BaseMirrorActivity<ActivityApiBinding>()`.
2. Update the left and right UI in `onCreate` via `mBindingPair.updateView { ... }`.
3. Collect and handle single clicks, double clicks, slides, etc. in `lifecycleScope` with `templeActionViewModel.state.collect { ... }`.

**Note**: The root layout intercepts touch events and delivers them to the Activity's `onTouchEvent` to ensure unified processing of temple gestures.

---
### 4.6 MirrorContainerView
**Package**: `com.ffalcon.mercury.android.sdk.ui.wiget`
A non-abstract, directly usable left and right mirrored container View (LinearLayout with left and right columns arranged horizontally). It does not depend on generics and dynamically binds the ViewBinding type via `bindTo`.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Used for quick integration of View-level binocular mirrored scenarios.
- **Threading**: `bindTo()` and subsequent View updates should be executed in the main thread.
- **Lifecycle**: `bindTo()` adds left and right subtrees to the container; it is usually recommended to call once per instance; avoid repeated stacking on repeated calls by yourself.

| Method                                             | Description                                                               |
|----------------------------------------------------|---------------------------------------------------------------------------|
| `bindTo(bindingClz: Class<B>): BindingPair<B>`     | Generates two left and right layouts using the specified ViewBinding class, adds them to the container, and returns `BindingPair<B>`. |

**Example (Inside FToast, FDialog, etc.)**
```kotlin
val toastView = MirrorContainerView(context).apply {
    val pair = bindTo(FfalconToastBinding::class.java)
    pair.updateView { tvToast.text = msg }
    make3DEffect(left.tvToast, right.tvToast, true, 15f)
}
```
---
### 4.7 BaseMirrorContainerView<B : ViewBinding>
**Package**: `com.ffalcon.mercury.android.sdk.ui.wiget`
A mirrored container base class that supports inheritance, specifies the ViewBinding via generics, and requires subclasses to implement `onInit()` for initialization. It holds `mBindingPair: BindingPair<B>` internally, with usage similar to `mBindingPair` in `BaseMirrorActivity`.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Used for custom View inheritance systems of binocular mirrored containers.
- **Threading**: Initialization and view updates must be performed in the main thread.
- **Lifecycle**: Initialization is triggered in the construction phase; avoid accessing unready external dependencies early in the construction (e.g., size information before attaching to the Window).

---
### 4.8 BaseMirrorFragment<B, H>
**Package**: `com.ffalcon.mercury.android.sdk.ui.fragment`
A Fragment-level mirrored base class that uses `HolderPair<B, H>` and `BindingPair<B>` to manage left and right Holders and Bindings. Suitable for implementing left and right symmetric UI and event synchronization in Fragments, with usage similar to Activity mirroring.

**Notes (Since / Threading / Lifecycle)**
- **Since**: Used for binocular mirrored encapsulation at the Fragment granularity.
- **Threading**: Fragment UI callbacks and `mBindingPair` updates should be executed in the main thread.
- **Lifecycle**: It is recommended to follow the pattern of creation in `onCreateView` and releasing View references in `onDestroyView` to avoid accessing Bindings beyond the View lifecycle.
---