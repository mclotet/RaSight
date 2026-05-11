---
name: mercury-checklist
description: MercurySDK Global Constraint Checklist.Applicable for pre-release Threading/Lifecycle review, risk verification and compliance self-check.
---

## 9. Global Constraint Checklist
This section abstracts the various `Since / Threading / Lifecycle` constraints in the preceding text into a pre-release checklist for easy review, testing and external release.

### 9.1 Threading Constraint Checklist
| Check Item                      | Applicable Objects                                                                                      | Mandatory Requirements                                                               |
|--------------------------------|---------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| SDK initialized once in main thread | `MercurySDK`                                                                                            | Call `init()` in `Application#onCreate`, avoid repeated initialization during runtime. |
| All View/Binding updates in main thread | `BindingPair`, `BaseMirrorActivity`, `MirrorContainerView`, `BaseMirrorContainerView`, `BaseMirrorFragment` | `updateView`, `setLeft`, `bindTo`, focus style updates, etc. all in the main thread.  |
| Gesture event stream consumption and UI modification in the same thread | `TempleActionViewModel`, `TempleAction`, `CommonTouchCallback`                                            | Events can be passed across threads, but the main thread must be used for UI modifications after consumption. |
| RecyclerView operations in main thread | `RecyclerViewFocusTracker`, `RecyclerViewSlidingTracker`, `StartSnapHelper`, `SimpleBindingAdapter`        | Including `notifyDataSetChanged`, scrolling, `dispatchTouchEvent` forwarding, `attachToRecyclerView`. |
| Dialog and Toast called only in main thread | `FDialog`, `FToast`                                                                                      | `show()` / `dismiss()` / Builder configuration and view updates all in the main thread. |
| Logging allowed across threads, avoid heavy logging in hot paths | `FLogger`                                                                                               | Reduce log volume on demand for high-frequency paths to avoid affecting input response. |

### 9.2 Lifecycle Constraint Checklist
| Check Item                  | Applicable Objects                                                           | Mandatory Requirements                                                          |
|----------------------------|------------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| Event collection bound to visible lifecycle | `TempleActionViewModel` (consumer side)                                      | Use `repeatOnLifecycle(STARTED/RESUMED)` to stop consumption when the page is invisible. |
| Automatic release of connection state monitoring | `MobileState`                                                              | Collect via controlled coroutines; ensure the internal `ContentObserver` is unregistered after the Flow completes. |
| Correct transfer of focus state during page switching | `IFocusable`, `FocusHolder`, `FixPosFocusTracker`, `FocusTracker`             | Switch `hasFocus` in a timely manner when pages/pop-ups are switched to avoid "invisible pages still responding to gestures". |
| Cleanup of dynamic focus items before destruction | `addFocusView`, `FocusViewHandle`                                           | Call `clearFocusView()` when no longer in use to avoid residual Views and references. |
| Follow-touch event stream started/stopped with focus | `RecyclerViewSlidingTracker`                                               | Stop forwarding original events when leaving the page or losing focus.            |
| Component resources released according to Android lifecycle | Camera/Sensor/GPS/Player scenarios                                           | Perform corresponding release in `onPause` / `onStop` / `onDestroy` to avoid power consumption and leaks. |

### 9.3 Release Review Checklist
```markdown
## Release Review Checklist
### Threading
- [ ] SDK initialized once in the main thread (call `MercurySDK.init()` in `Application#onCreate`)
- [ ] All View/Binding updates executed in the main thread (`updateView` / `setLeft` / `bindTo`)
- [ ] UI modifications after gesture event stream consumption executed in the main thread
- [ ] RecyclerView-related operations in the main thread (scrolling, refresh, event forwarding, SnapHelper attachment)
- [ ] Dialog and Toast called only in the main thread (`show` / `dismiss` / Builder configuration)
- [ ] Log volume reduced for high-frequency paths to avoid affecting input response

### Lifecycle
- [ ] Gesture event collection bound to visible lifecycle (`repeatOnLifecycle(STARTED/RESUMED)`)
- [ ] `MobileState` connection monitoring automatically released after page leave (no unbounded hanging collection)
- [ ] Focus state correctly transferred during page/pop-up switching (avoid invisible pages responding to gestures)
- [ ] Dynamic focus items cleaned up before destruction (call `clearFocusView()`)
- [ ] Follow-touch event stream started/stopped with focus (stop forwarding after leaving the page or losing focus)
- [ ] Camera/Sensor/GPS/Player resources correctly released in `onPause/onStop/onDestroy`
```
---
