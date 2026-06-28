# TGAM Mac EEG Run Package

This folder contains the patched NeuroSky Mac Developer Tools files needed to build and run the EEG Algo SDK sample GUI on macOS with the TGAM headset/board.

## What is included

- `OSX Developer Tools 3.3/EEG_Algo_SDK_OSX/Algo SDK Sample/`
  - The patched Xcode GUI project.
  - `AlgoSdk.framework`, the closed-source NeuroSky algorithm SDK.
  - `TGStreamMac.framework`, the official newer Stream SDK framework copied from `Stream SDK for Mac/lib` so the app can open an explicit serial port path.
  - `CorePlot.framework`, used by the sample GUI for graphs.
- `OSX Developer Tools 3.3/Stream SDK for Mac/lib/TGStreamMac.framework`
  - The source location of the newer official `TGStreamMac.framework` used by the GUI.
- `docs/`
  - `EEG_Algo_SDK_README.txt`
  - `Algo_SDK_Sample_README.txt`
  - `eeg_algorithm_sdk_for_osx_development_guide.pdf`
  - `eeg_algorithm_descriptions_free_version.pdf`
  - `Stream_SDK_README.txt`
  - `StreamSDKForMac.pdf`
  - `MDT_OSX_Instruction.pdf`
  - `ApplicationStandards.pdf`
  - `EULA.pdf`


## Requirements

- macOS with Xcode installed.
- Rosetta on Apple Silicon Macs, because the bundled NeuroSky frameworks are Intel-only.
- TGAM headset/board visible as a serial device under `/dev`.

Check serial devices with:

```sh
ls /dev/tty.* /dev/cu.*
```

In our working setup, the TGAM device path was:

```text
/dev/tty.usbmodem8D71129C48571
```

## Build and run in Xcode

1. Open:

```text
OSX Developer Tools 3.3/EEG_Algo_SDK_OSX/Algo SDK Sample/Algo SDK Sample.xcodeproj
```

2. Select the `Algo SDK Sample` scheme.
3. Build and run.
4. In the app, click `Connect Headset`.
5. Select algorithms such as `Attention`, `Meditation`, `Eye Blink`, and `EEG Bandpower`.
6. Click `Set Algos`.
7. Click `Start`.
8. Use the segment control to view `EEG Raw`, `Frequency`, or `EEG Bandpower`.

## Build and run from Terminal

From this package folder:

```sh
env HOME=/private/tmp/tgam_xcode_home CFFIXED_USER_HOME=/private/tmp/tgam_xcode_home \
xcodebuild -project 'OSX Developer Tools 3.3/EEG_Algo_SDK_OSX/Algo SDK Sample/Algo SDK Sample.xcodeproj' \
-scheme 'Algo SDK Sample' -configuration Debug -destination 'platform=macOS' \
-derivedDataPath /private/tmp/tgam_xcodebuild/AlgoSDKProjectSettings build
```

Then launch:

```sh
open -n '/private/tmp/tgam_xcodebuild/AlgoSDKProjectSettings/Build/Products/Debug/Algo SDK Sample.app'
```

## If the serial device path changes

Edit this line in:

```text
OSX Developer Tools 3.3/EEG_Algo_SDK_OSX/Algo SDK Sample/Algo SDK Sample/ViewController.mm
```

```objc
static NSString * const kTGAMSerialPortPath = @"/dev/tty.usbmodem8D71129C48571";
```

Replace it with the serial path shown by your Mac.

## Notes on algorithms

`AlgoSdk.framework` is a compiled NeuroSky binary. Its Attention, Meditation, Eye Blink, and Bandpower internals are not included as editable source code.

The sample app feeds raw EEG and poor-signal values into `AlgoSdk.framework`. The SDK then returns algorithm callbacks such as `attAlgoIndex`, `medAlgoIndex`, `eyeBlinkDetect`, and `bpAlgoIndex`.

This package also contains a custom raw-EEG blink detector added in `ViewController.mm`. It looks for large raw-signal transients using a rolling peak-to-peak range, instantaneous slope, and a short refractory period. That detector is editable source code and is separate from NeuroSky's closed-source blink algorithm.

## Summary of patch purpose

The original official Mac Developer Tools 3.3 package is old. These patches make the sample easier to run on modern macOS:

- repaired framework symlinks;
- forced x86_64 build for old Intel-only frameworks;
- updated deployment metadata for modern Xcode;
- added Bluetooth privacy text;
- replaced the old Algo sample `TGStreamMac.framework` with the newer official Stream SDK copy;
- routed the app to an explicit TGAM serial port;
- added diagnostics for raw callbacks, connection state, and blink events;
- widened the raw EEG graph range;
- added visible blink status and a custom blink detector.
# EEG_sdk
