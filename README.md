# RaSight: RayNeo Mercury SDK Demo Project

RaSight is a modernized demonstration project for the **RayNeo Mercury SDK**, specifically optimized for the **RayNeo X3 Series** AR glasses. It showcases core AR capabilities including binocular 3D rendering, gesture-based interaction, and AI-driven "Vibe Coding" integration.

## 🚀 Key Features

- **Binocular 3D UI**: Symmetric layout management using `BindingPair` and `BaseMirrorActivity` with adjustable parallax effects.
- **Temple Touch Interaction**: Unified gesture handling (Single Tap, Double Tap, Slide) via `TempleActionViewModel`.
- **Advanced Focus Management**: Head-tracking and touch-based focus tracking for standard views and complex lists.
- **Vibe Coding Ready**: Integrated AI Skill specifications in `.cursor/rules` for AI-assisted development.
- **Modern Android Stack**: Built with AGP 9.2.1, Kotlin 2.2.10, and targeting Android API 34.

## 🛠️ Project Structure

- `app/`: Main application module containing the demo activities.
- `app/libs/`: Contains the proprietary RayNeo Mercury AAR and IPC SDKs.
- `.cursor/rules/`: AI Skill specifications for natural language development using Vibe Coding.
- `docs/`: Original SDK API references (CN).

## 📋 Prerequisites

- **IDE**: Android Studio Ladybug (2024.2.1) or higher.
- **Hardware**: RayNeo X3 Pro AR Glasses.
- **SDK**: Android API 34 (UpsideDownCake).

## ⚙️ Getting Started

### 1. Connection
Ensure your glasses are connected to your PC via a high-quality USB-C data cable.

### 2. Developer Mode
On your glasses (or the connected controller device):
1. Go to **Settings > About**.
2. Tap **Build Number** 7 times to enable Developer Options.
3. In **Developer Options**, enable **USB Debugging**.

### 3. Build & Run
```bash
./gradlew assembleDebug
```
Deploy to the device via Android Studio. The system will automatically recognize the app as a RayNeo-compatible application due to the required `<meta-data>` in the `AndroidManifest.xml`.

## 🤖 Vibe Coding with AI
This repository is optimized for AI IDEs like **Cursor**, **Windsurf**, or **GitHub Copilot**. 

Because the Mercury Skill specifications are included in `.cursor/rules`, you can prompt your AI agent with commands like:
- *"Create a new binocular screen with a 3D parallax effect on the center image."*
- *"Implement a scrollable list that handles temple-slide gestures."*

## 📄 License
This project uses the RayNeo Mercury SDK. Please refer to the [official RayNeo developer documentation](https://open.rayneo.com) for licensing terms regarding the SDK components.
