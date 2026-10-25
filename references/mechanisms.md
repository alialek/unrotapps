# Discovering and limiting attention-retention mechanics

These are portable investigation methods, not ready-made patches. Search strings are leads; a matching string alone does not establish what the code does.

## Algorithmic feed → chronological/following

**Find:** feed selectors, mode enums, saved preferences, request builders, cursors and pagination sources, response filters, and mode-change handlers. Search leads: `following`, `favorites`, `friends`, `chronological`, `recent`, `ranked`, `blended`, `for_you`, `recommendations`, `suggested`, `feed_type`, `pagination_source`, `last_selected_feed_type`.

**Check semantics:**

- Following is different from mutual friends; favorites may be a separate list.
- “Recent” can refer to interaction time rather than publication time.
- Chronological order does not imply the absence of ads or recommendations.
- A UI label or saved preference does not prove the request uses that mode.
- A mode may reset on restart or through a server experiment.

**Preferred intervention:** use an existing supported mode consistently across the selector, preference, initial request, and subsequent pages. Reset incompatible cursors and that mode's cache when switching, without clearing all app data.

Describe local filtering as filtering. Sorting a returned page cannot guarantee global chronological order: the server may have omitted posts. Discover actual endpoints and enum values. Avoid collecting an unnecessary social graph just to define “friends.”

**Validate:** authors belong to the chosen group; timestamps are ordered across several pages; the mode survives a cold start; cursors do not mix feeds. Check ads and recommendations separately if changing them was requested.

## Endless scroll → Load more

**Find:** `next_cursor`/`max_id`/`has_more`, `loadMore`/`fetchNextPage`/`prefetch`, scroll listeners, viewport/visibility callbacks, last-item observers, loader policies, footers, and retry handlers. Include loading after cache restoration, refresh, and empty responses.

Distinguish:

1. Initial load.
2. Explicit refresh.
3. Automatic next-page loading.
4. Manual next-page loading or retry.

Preserve the first two unless asked otherwise. Disable the third and retain the fourth.

**Useful intervention point:** a pagination policy or request with a separate permit for one explicit user action. Guard the paths that actually initiate the next page, including prefetch. Scrolling within already loaded content should remain available.

Consume permits predictably. Clear them after errors/refusal; prevent reuse on another surface or a later automatic attempt. Repeated taps while loading must not create duplicate requests. Preserve cursors, deduplication, retry, and the terminal “no more items” state.

Issue the permit in the actual click handler, not a shared retry/load-more method that the app can call itself. Check independent eager-prefetch requests and the insertion of prefetched content into the adapter: blocking one HTTP path may leave other ways to extend the feed.

Reuse an existing footer/LoadMoreButton where possible. Recheck `isLoading`: in an automatic feed, it may effectively mean “another page exists,” leaving a permanent spinner after automatic loading is disabled.

Style the button with the app's design-system components: theme, typography, dimensions, focus, accessible name, and disabled/loading states. Preserve its click handler when replacing the visual component.

Keep changes scoped. A global RecyclerView/IntersectionObserver/fetch block can break messages, comments, and unrelated lists. Home and Explore often have different loaders.

**Validate:** repeated swipes at the boundary leave the same end visible; no pagination occurs before a tap; one tap adds one logical page; the next boundary requires another tap. Include rapid gestures with time for momentum to settle, across multiple boundaries. Check offline/retry and a rapid double tap when request handling changes.

## Short video and automatic transitions

Investigate these independent mechanisms:

- Entry through a tab, card, message, or deep link.
- A gesture to the adjacent video.
- Fake drag, smooth scrolling, accessibility actions, or other programmatic transitions.
- Auto-next at playback end.
- Pull-to-refresh that replaces the current video.

Hiding a tab is insufficient for full blocking. Another controller may re-enable input after `setUserInputEnabled(false)`; a parent may intercept gestures even when the child pager is blocked.

When viewing a specific video remains allowed, preserve initial positioning on the supplied media ID. Limit subsequent transitions and refresh rather than every `setCurrentItem` call.

Distinguish taps, vertical motion, and horizontal carousels. Cancel a child's gesture to prevent accidental like/follow actions on ACTION_UP. Modify shared UI components only with a reliable scope check for the intended viewer; check neighboring screens too.

In advanced mode, when redirecting a tab to the user's chosen app, check the package, package visibility, and missing-target behavior. A failed launch must not silently open the restricted feed.

## Push notifications

**Resolve categories from context:** recommendations, new posts, likes/followers, messages, calls, or security. Respect an explicit request for all categories. “Unwanted notifications” alone does not authorize disabling messages or security alerts.

**Settings first:** inspect category settings within the app, then per-app OS settings and notification channels. Investigate silent delivery, sound/banner/badge controls, or schedules where supported and relevant. Check settings, API, permission, and command availability for the current OS version.

**Code search leads:** `NotificationManager`/`notify`, `NotificationChannel`, `FirebaseMessagingService`/`onMessageReceived`, push providers, category/type/action, channel IDs, notification builders, and deferred/local scheduling. On other platforms, investigate their notification APIs, service workers, or desktop notification bridges.

Separate push receipt, data processing, system notification display, badges, and in-app banners. A push can update a chat or other useful state without displaying an alert.

For selective client filtering, prefer gating display by a confirmed category/channel. Identify useful side effects before stopping an entire message handler. Keep shared Play Services, network domains, and device push infrastructure intact. Re-signing or restarting may recreate channels; verify that separately.

Ordinary notification suppression does not require extracting or deleting push tokens or credentials. Sending people test messages requires authorization for that scenario.

**Validate:** an incoming event in a blocked category produces no display; an allowed category still works; settings survive restart. No incoming event means no evidence of filtering. A directly generated test notification checks display only, not the delivery chain. Account for notifications already present.

## Mode boundaries

In light mode, retain buttons and destinations. Limit transitions to the next item while preserving specific content access and ordinary actions within it. Manual pagination limits loading, not movement within a loaded page.

In advanced mode, verify the original handler, preloading, and alternative entry points within the agreed scope are disabled. Reassignment uses parameters from the current task. Reminders use a local dialog with dismissal and no automatic continuation into the disabled feature.
