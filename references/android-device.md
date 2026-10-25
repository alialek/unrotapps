# Android package sources, connection, and installation

## Prepare the phone

Install the official [SDK Platform Tools](https://developer.android.com/tools/releases/platform-tools), or use an existing SDK. Check `adb version`; if several copies are installed, choose one explicit adb path. Signature and alignment checks require SDK Build Tools (`apksigner`, `zipalign`) separately.

On the phone, enable Developer options (usually by tapping Build number seven times), then USB debugging. Connect with a data-capable cable, unlock the phone, and approve your computer's RSA key. The user enters their PIN, password, and biometrics on the phone. macOS needs no additional USB driver; Windows may need an OEM driver. See [device setup](https://developer.android.com/studio/run/device) and [Developer options](https://developer.android.com/studio/debug/dev-options).

```sh
adb devices -l
```

Select SERIAL explicitly for subsequent commands. `device` means connected, not necessarily unlocked; `unauthorized` requires approval on the phone; `offline` requires restoring the connection. If the list is empty, check the cable, port, and USB debugging. Do not bypass the lock or enable root/bootloader unlocking for this task. With several devices, do not select the first automatically.

## Where to obtain APKs

Prefer the user's official installed copy from Google Play or a download supplied by the publisher. Extract its base and splits without using a third-party APK catalog. Extraction alone does not establish official provenance: record the source and compare the certificate against an independent trusted reference. Do not promise a universal direct Google Play APK URL.

For APK/APKS files from another source, first follow the [provenance checks](android-apk.md). Keep splits version-compatible. An installed set is device-specific and may omit other ABIs, languages, or dynamic modules that have not been downloaded.

## Extract an installed package

SERIAL, PACKAGE, and paths below are parameters to fill in, not ready-to-run values. Identify the chosen app's package without unnecessarily collecting the user's whole app inventory. For a work profile, determine the intended user ID and supported `pm` syntax separately.

```sh
adb -s SERIAL shell pm path PACKAGE
adb -s SERIAL shell dumpsys package PACKAGE
```

Record version information and paths. Every `package:/...apk` line names a separate file. Strip only the `package:` prefix and `pull` **every** returned file, including base, configuration, and feature splits. Example for one file:

```sh
adb -s SERIAL pull '/data/app/ACTUAL_PATH/base.apk' './input/base.apk'
```

Create a separate input directory first. Preserve filenames, check each operation's exit status, log “file N of M,” and compute SHA-256. Never pass untrusted device output into `eval` or an interpolated shell command; use subprocess argument arrays for automation. Recheck the version and paths afterward: an automatic update during extraction makes the set unreliable. Extract a consistent set again if that happens.

Verify every APK with `apksigner verify --verbose --print-certs FILE.apk`. Extracting APKs does not copy app data, passwords, sessions, or the publisher's signing key. If device policy prevents reading, explain the limitation and use an authorized source instead of bypassing device management.

## Install the prepared result

Check package, versionCode, certificate, and split compatibility first. A modified build uses its own signing key and generally cannot update a publisher-signed original. Preserving the original requires a correctly isolated clone with a different package; changing its display name alone is insufficient. Updating an existing clone requires its previous key. See [android-apk.md](android-apk.md).

For a standalone APK:

```sh
adb -s SERIAL install -r './output/app.apk'
```

For a split set, list **all required files explicitly**. These filenames are examples; determine the actual set from the input:

```sh
adb -s SERIAL install-multiple -r './output/base.apk' './output/split_config.arm64_v8a.apk' './output/split_config.xxhdpi.apk'
```

Do not pass an `.apks` container directly to `adb install`: inspect its format and select a compatible, consistent set first. Avoid wildcards that can include unrelated APKs. `-r` updates a compatible package while preserving data, although app migrations may still occur. Command reference: [Android Debug Bridge](https://developer.android.com/tools/adb).

On signature/version conflicts, do not automatically run `uninstall`, `pm clear`, or a forced downgrade. Diagnose the cause while preserving the original, data, and working artifact. Lower versionCode installation may be prohibited; one APK alone cannot guarantee data recovery.

The user approves any additional USB installation prompt on the phone. Manual installation through a file manager may require “Install unknown apps” permission for that installer; this is separate from USB debugging. Do not disable device protection or grant broad app permissions to make installation succeed.

After successful installation, confirm the installed package/version; extract the installed base and compare hashes when needed. Launch and perform [behavioral validation](validation.md), including user sign-in. Installation `Success` does not establish working login. Explain how to disable USB debugging and revoke computer authorization afterward if desired; change those settings only when requested.
