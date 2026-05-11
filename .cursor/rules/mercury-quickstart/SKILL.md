---
name: mercury-quickstart
description: MercurySDK Quick Start & Initialization Flow.Applicable for pre-integration checks, Manifest configuration, Application initialization, and implementation of basic interaction conventions.
---

## 2. Quick Start
### 2.1 Prerequisites (From RayNeo X3 ARSDK Document)
To ensure the normal operation of the APIs in this document, it is recommended to meet the following basic integration requirements first:
1. Enable ViewBinding (the SDK is encapsulated based on ViewBinding):
```groovy
buildFeatures {
    viewBinding = true
}
```
2. Add the following to the `application` node in the main module's `AndroidManifest.xml`:
```xml
<meta-data
    android:name="com.rayneo.mercury.app"
    android:value="true" />
```
The application may not be displayed in the glasses Launcher if this configuration is missing.

### 2.2 Initialize the SDK
Call the initialization once in `Application#onCreate`:
```kotlin
// In custom Application subclass
override fun onCreate() {
    super.onCreate()
    MercurySDK.init(this)
}
```
**Class**: `com.ffalcon.mercury.android.sdk.MercurySDK`

| Method                               | Description                                                                 |
|--------------------------------------|-----------------------------------------------------------------------------|
| `init(application: Application)`     | Initializes the SDK with the current Application, must be called before using other SDK capabilities. |
| `mApplication: Application`          | Read-only. Get the Application instance through this property after initialization. |

**Notes (Since / Threading / Lifecycle)**
- **Since**: Corresponding MercurySDK version line of the current repository (`v0.2.4` series).
- **Threading**: `init()` is recommended to be called only once in the main thread during the application startup phase.
- **Lifecycle**: It is recommended to complete initialization in `Application#onCreate`; repeated initialization will not report an error, but frequent calls during runtime are not recommended.

### 2.3 Recommended Page Interaction Conventions (From RayNeo X3 ARSDK Document)
- Focus switching: Usually use forward/backward slide.
- Trigger current focus action: Usually use single click.
- Page return: It is recommended to exit the current page with double click (consistent with Demo and SDK examples).
---
