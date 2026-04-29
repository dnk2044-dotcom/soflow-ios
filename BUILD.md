# SOFLOW FOTA — Cordova iOS build guide

This is a port of the **JMTOtaActivity / OtaActivity** screen from the
SO ONE-PLUS Android APK. The original was a Cordova WebView that ran
a 325-line JavaScript app (`www/js/index.js`) using
`cordova-plugin-ble-central`. That plugin already has a CoreBluetooth
implementation for iOS, so this part of the app rebuilds for iOS with
zero code changes.

**Caveat:** this only covers BLE firmware updates. The full SO ONE-PLUS
app (login, dashboard, throttle/speed settings, factory tools, TPMS,
diagnostics, unlock records) is **native Java/Kotlin** and is NOT
covered here — see `../SOFLOW-iOS-Native/` for that side.

## Prerequisites

You need:
- A **Mac** running macOS 13+ (Ventura) — Xcode does not run on Windows.
- **Xcode 15+** (free from the Mac App Store, ~10 GB).
- **Node.js 18+** (https://nodejs.org/).
- **CocoaPods**: `sudo gem install cocoapods`.
- **Cordova CLI**: `npm install -g cordova`.

Plus one of the following code-signing options:

**Option 1 — Free 7-day sideload** (no money, but app expires every 7 days):
- An Apple ID (you already have one — dean.karpouchtsis@icloud.com).
- Just sign the app with your personal team in Xcode. Re-sign every 7 days.

**Option 2 — Apple Developer Program** ($99/year):
- Sign up at https://developer.apple.com/programs/enroll/
- Install gives you 1-year provisioning, TestFlight, ad-hoc distribution.

**Option 3 — AltStore / SideStore**:
- AltStore is the cleanest free option for permanent-ish sideloading.
- Install AltServer on your Mac/PC, AltStore on iPhone, build the .ipa here,
  drag into AltStore. AltStore auto-resigns every 7 days while your phone
  is on the same Wi-Fi as the AltServer machine.

## Build steps (on your Mac)

1. Copy this folder to your Mac (e.g. via iCloud Drive — you're already
   working in `iCloudDrive/SOFLOW/ios-rebuild/SOFLOW-FOTA-Cordova/`).

2. In Terminal:

       cd ~/Library/Mobile\ Documents/com~apple~CloudDocs/SOFLOW/ios-rebuild/SOFLOW-FOTA-Cordova
       # (or wherever the path resolves on your Mac)
       npm install
       cordova platform add ios
       cordova prepare ios
       cordova build ios --release

3. Open the generated Xcode project:

       open platforms/ios/SOFLOW\ FOTA.xcworkspace

4. In Xcode:
   - Set the **Team** to your Apple ID (Signing & Capabilities tab).
   - Set the **Bundle Identifier** to something unique like
     `com.dean.soflow.fota` (already in config.xml).
   - Plug in your iPhone 16 via USB (or use Wi-Fi if paired).
   - Trust the computer on the phone.
   - Pick your phone as the run target (top toolbar) and hit **▶ Run**.

5. On your iPhone, the first time you run a self-signed app:
   - Settings → General → VPN & Device Management → trust your developer
     profile. Then re-launch the app.

## What this app does

- Scans for nearby BLE devices (no name filter — see "Limitations").
- Connects, requests MTU 517, reads the FOTA service (UUID `2600`) with
  control characteristic `7000` and data characteristic `7001`.
- Lets you pick a `.bin` firmware image and a signature file from your
  phone, computes SHA-256 of the firmware, then runs the SOFLOW FOTA
  protocol: SIGNATURE → DIGEST → START_REQ → START_RSP → write image
  in NEW_SECTOR chunks → INTEGRITY_CHECK_REQ/RSP.

## Limitations

- The original Android version disables a name filter
  (`if(device.name.search("SFS")!=-1)` is commented out). All BLE
  devices show up — pick the one named `HIBOY*` / `SOFLOW*` /
  `SO ONE-PLUS*`.
- iOS file picker is more restricted than Android. You may need to put
  the `.bin` and signature file in the Files app first (iCloud or
  On My iPhone) before they're selectable.
- This is **not** the full SO ONE-PLUS app — only the firmware updater
  piece. Login, ride dashboard, etc. are not in this build.

## Troubleshooting

- **"Code signing certificate not found"** in Xcode → Preferences →
  Accounts → add your Apple ID, then refresh manual provisioning.
- **App crashes immediately on launch with no Bluetooth prompt** →
  check `NSBluetoothAlwaysUsageDescription` is present in Info.plist
  (config.xml should set it; if not, edit the plist directly in Xcode).
- **MTU stays at 23** — iOS auto-negotiates MTU and ignores explicit
  `requestMtu()`. The plugin call is a no-op on iOS but does not error.
- **Self-signed builds expire after 7 days** — re-build, or use AltStore.
