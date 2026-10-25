---
name: unrotapps
description: Reduce doomscrolling and distracting app mechanics such as algorithmic recommendations, infinite feeds, short-video paging, and unwanted notifications through settings or targeted client changes. Use for requests to remove Reels scrolling, add Load more instead of endless scrolling, show followed accounts, or make similar changes to an existing app.
---

# Unrot Apps

Preserve the app functions the user values while limiting specific attention-retention mechanics. Keep the work focused on the requested behavior.

## Establish scope

Identify from the request and context:

- App, platform, version, and build source.
- Specific surfaces: home feed, Explore, search results, profile, media viewer, or messages.
- Desired behavior: chronological/following feed, manual next page, no next video, or selected notification categories.
- Functions to retain, such as messages, comments, calls, and security notifications.

Clarify only material unknowns while continuing independent analysis. “The feed” does not mean every RecyclerView in the app. Investigate Explore separately if the user adds it to scope.

A previous case does not authorize installing another build, removing the original, or changing a new user's settings. Follow the current request without asking again for an already authorized update.

## Choose a mode and an approach

Infer the mode from the current request; clarify it when an unresolved choice affects the result. Apply it only to the agreed surfaces.

- **Light:** retain functionality, buttons, and content entry points; disable continuous scrolling. In a viewer, preserve opening a specific item while blocking subsequent swipes and bypasses through auto-next/refresh. In a feed, allow scrolling within loaded content and require **Load more** for the next page. Keep buttons and their destinations intact.
- **Advanced:** disable buttons for the selected functionality and its other entry points when full removal is requested. Optionally reassign buttons to another app or a personal reminder. The target package and reminder text are task parameters with no personal defaults. Hiding one tab does not prove the feature is inaccessible.

Feed order, feed membership, and push categories are separate options; neither mode authorizes changing them automatically. For a reminder shown on tap, prefer a local screen/dialog with an explicit dismiss action, without new push notifications, timers, or a hidden transition into the disabled feature.

1. Inspect relevant app and OS settings: feed order, following/favorites, autoplay, and notification categories.
2. Use an existing setting when it meets the request. Verify persistence after restart and actual behavior, beyond the switch label.
3. If a client change is needed, trace **trigger → controller → state/request → rendering**. Change the narrowest suitable point.
4. Explain server-side limits when the required mode or data is unavailable. Sorting one returned page cannot establish global chronological order.
5. Preserve original source/packages and a reproducible description of changes. Log stages, progress, and a log location for long processes.

Read the resources relevant to the task:

- [Mechanism discovery and intervention](references/mechanisms.md).
- [Android APK analysis and rebuilding](references/android-apk.md).
- [Android package sources, extraction, ADB, and installation](references/android-device.md).
- [Device validation and evidence](references/validation.md).

APK procedures apply to Android. For web, desktop, or iOS, use available settings, source code, or authorized client-extension mechanisms; patching and installation capabilities do not transfer automatically between platforms.

## Version-specific subskills

For Android Instagram, read the [452.1.0.53.64 subskill](subskills/instagram-452-1-0-53-64/SKILL.md), then match the input APK fingerprint.

Obfuscated class names, method numbers, resource IDs, and feature flags **are not general rules**. A different version or fingerprint requires fresh investigation by behavior. Use the old map as a lead, never as permission to patch a matching filename blindly.

When adding a subskill, record exact applicability, input fingerprint, mechanism map, minimal changes, build invariants, tested scenarios, and unverified boundaries. Avoid duplicating the general guidance.

## Completion criteria

Confirm the requested behavior on the actual surface, retained functions, and installed build if installation was in scope. Without a device, finish the static work but report device testing as pending.

Separate **device-tested**, **confirmed in code**, and **hypothesis/untested** in the report. Name changed surfaces and remaining limitations. Claims about all feeds or complete safety require evidence beyond a limited set of checks.
