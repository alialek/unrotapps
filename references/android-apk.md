# Android APK analysis, rebuilding, and updates

See [android-device.md](android-device.md) for connection setup, package sources, extraction of all installed splits, and installation commands. APKs can be obtained from the device without downloading from a third-party catalog.

## Input artifact

Before editing, record container and base APK SHA-256, package name, versionName/versionCode, ABI/minSdk/targetSdk, split set, and signing certificates. Work on a copy in a separate directory; keep the input APK/APKS unchanged.

Verify each required split with `apksigner`. Matching base/split certificates establish consistency, not official provenance by themselves. Compare against an independent trusted fingerprint and account for signing lineage/key rotation. A re-signed modified APK no longer carries the publisher's signature.

Signature verification is not a complete malware audit. Static domains do not prove credential exfiltration: a domain may be constructed dynamically or used by a legitimate SDK. If an audit is requested, distinguish provenance, static analysis, and observed network behavior. Uploading private APKs/logs to external services requires authorization.

## Investigation

Start with manifest/resources, strings, original-name markers, navigation points, and existing policies. Use decompiler output to understand behavior, then check changes against DEX/smali and callers/callees.

On case-insensitive filesystems, names such as `00tU` and `00TU` may gain `.1`/`.2` filename suffixes. The `.class` directive is authoritative.

Map the specific path: surface → entry point → automatic trigger → manual trigger → request/state → UI. Record alternatives selected by feature flags. Avoid broad flag changes.

## Cloning and data preservation

Choose an update to an existing personal clone or a separate package according to the task. A differently signed build generally cannot update the official app in place. Do not resolve a signature conflict by deleting the original.

For a clone, investigate package identity, authorities, custom permissions, content URIs, task affinity, component identities, and account services. Blind replacement of every package string can break Java/native class names and external URLs that require the original value. Use an exact mapping and correctly reserialize DEX and binary XML.

Some apps read `ApplicationInfo.labelRes` directly: a string launcher label may not replace the required resource reference. Preserve necessary resource IDs and change display names separately. AccountAuthenticator isolation is app-specific, not a universal step.

Login, signatures, app links, and Play Integrity can depend on official package identity. Do not promise every clone can sign in. For network failures, inspect per-app VPN configuration: a new package/UID may be absent from its allowlist. Treat this as a diagnostic hypothesis.

Store the personal signing key outside the skill/repository and keep passwords out of output. Use the clone's existing key for updates. Exclude secrets, sessions, and private APKs from the distributed skill.

## Build

- Change only necessary classes/resources; avoid rebuilding everything without a reason.
- Keep density, ABI, and language splits version-compatible. Missing language modules can prevent startup; a module from another version requires separate investigation.
- Treat split merging as a separate transformation. Check manifest, resources, assets, and native libraries after merging.
- Check DEX reference limits, registers/invoke ranges, checksums, resource references, exception tables, and loader metadata.
- Remove invalidated old signatures/source stamps/code-transparency metadata where applicable; do not present them as provenance for the modified build.
- Follow current build-tools/device requirements for ZIP alignment, native library alignment/compression, signatures, and split installation.
- Inspect local tools and SDK flags first. Resolve unfamiliar or changed APIs through current primary documentation.

Log stages and processed class/entry counts. On a memory-constrained Mac, bound heap usage and run heavy operations sequentially. MPS does not accelerate smali/ZIP/DEX work.

## Verification and installation

Compare classes with the input or a previously verified build using an explicit allowed change set. Serialization differences need evidence of equivalence; do not expand the allowlist just to pass. Pay particular attention to exception handlers.

Check archive CRC, DEX checksums, loader metadata, signing, alignment, package/splits, and unchanged libraries/assets. For a small follow-up iteration, verify the changed DEX and require all others to match the verified baseline byte-for-byte.

Check `adb devices` and select the serial explicitly in every device command. Installing a compatible, correctly signed clone with `-r` preserves its data, although the app may perform migrations. Avoid `uninstall`/`pm clear` as routine error recovery. Passwords and biometrics stay on the user's device.

Wait for the installation exit status before launching. Confirm the installed package and, where needed, compare the installed base SHA-256 with the local output. Resetting compiled-code caches is a justified diagnostic operation only, not a default response to a failed check or a substitute for preserving user data.
