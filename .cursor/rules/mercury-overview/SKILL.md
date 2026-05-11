---
name: mercury-overview
description: MercurySDK Overview & API Index.Applicable for scenarios that require a quick understanding of the SDK's capability boundaries, core type entry points, and module positioning.
---

## 1. Overview
MercurySDK provides the following capabilities for RayNeo AR glasses:
- **Binocular mirrored layout**: Symmetric and synchronous update of left and right eye views, with support for 3D parallax effects.
- **Temple touch control**: Maps temple touch events to unified gestures (single click, double click, slide, etc.) for easy operation on touchless devices.
- **Focus management**: Switches focus via temple sliding/clicking, and handles focus and tracking in lists and pop-ups.

All **package names** in this document are rooted at `com.ffalcon.mercury.android.sdk`.

### 1.1 Class & Function Signature Summary (Reference Index)
The table below is for quick location of commonly used types and entry points, following the style of Android official Reference with "class overview + core signatures".

| Type                           | Package Path          | Core Signatures (Excerpts)                          | Function                                          |
|--------------------------------|-----------------------|-----------------------------------------------------|---------------------------------------------------|
| `MercurySDK`                   | `...sdk`              | `init(application: Application)`                    | SDK initialization entry point                    |
| `MobileState`                  | `...sdk.api`          | `isMobileConnected(): Flow<Boolean>`                | Mobile phone connection state monitoring          |
| `BindingPair<B>`               | `...sdk.core`         | `updateView { }` / `setLeft { }` / `checkIsLeft(...)`| Left-right layout mapping and synchronous update  |
| `make3DEffect`                 | `...sdk.core`         | `make3DEffect(leftView, rightView, enable, parallax)`| Binocular 3D parallax configuration               |
| `make3DEffectForSide`          | `...sdk.core`         | `make3DEffectForSide(view, isLeft, enable, parallax)`| Monocular 3D parallax configuration               |
| `BaseMirrorActivity<B>`        | `...sdk.ui.activity`  | `abstract class BaseMirrorActivity<B : ViewBinding>`| Activity-level binocular base class               |
| `FToast`                       | `...sdk.ui.toast`     | `show(...)` / `showCustom(...)`                     | Binocular Toast                                   |
| `FDialog.Builder<T>`           | `...sdk.ui.dialog`    | `setContentView(...)` / `setEventHandler(...)`      | Binocular Dialog builder                          |
| `TempleAction`                 | `...sdk.touch`        | `sealed class TempleAction`                         | Gesture semantic model                            |
| `TempleActionViewModel`        | `...sdk.touch`        | `state: SharedFlow<TempleAction>`                   | Gesture event stream distribution                 |
| `FocusHolder` / `FocusInfo`    | `...sdk.ui.util`      | `addFocusTarget(...)` / `currentFocus(...)`         | Universal focus item management                   |
| `FixPosFocusTracker`           | `...sdk.ui.util`      | `handleFocusTargetEvent(action)`                    | Fixed focus item switching logic                  |
| `RecyclerViewFocusTracker`     | `...sdk.ui.util`      | `handleActionEvent(it, block)`                      | Moving focus item list tracking                   |
| `RecyclerViewSlidingTracker`   | `...sdk.ui.util`      | `observeOriginMotionEventStream(...)`               | Fixed focus + follow-touch scrolling              |
| `StartSnapHelper`              | `...sdk.util`         | `StartSnapHelper(offset2Start)`                     | List start snap                                   |
| `DeviceUtil`                   | `...sdk.util`         | `isX3Device(): Boolean`                             | RayNeo X3 device judgment                         |
| `FLogger`                      | `...sdk.util`         | `d(...)` / `i(...)` / `e(...)`                      | Unified SDK logging                               |
---