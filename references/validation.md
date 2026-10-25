# Behavioral validation

## Reliable experiments

Before each scenario, establish **which screen is actually open** using the UI tree, package/activity, and distinctive elements. A video card in the home feed is not evidence that a separate Reels viewer is open.

Obtain coordinates from the current UI tree. Use screenshots for visual assessment. If video prevents stable hierarchy inspection, try another available inspector. An inspector failure is not proof of an app failure.

Change one condition at a time. For a video test: open the item, record its ID/author/caption, send one gesture, and inspect again. Author plus caption is approximate identity; prefer a media ID when available.

Check separately whether a test tool launches the app, restores an Activity, or changes state between flows. An unexpected result alone does not prove the tool reopened the screen. Direct ADB input events with a separate read-only inspector can isolate the experiment.

If simultaneous user input is suspected, briefly ask the user to leave the screen untouched. Confirm state after each gesture instead of only after a long batch.

## Minimum matrix

Validate the chosen mode. **Light:** original buttons and specific content access still work, loaded content remains scrollable, continuous consumption stops, and destinations remain intact. **Advanced:** disabled buttons and agreed alternative entries cannot open the feature; reassigned buttons perform the chosen action only. A reminder shows the supplied text and dismisses without entering the blocked section. Check missing target apps separately.

| Change | Evidence of success | Possible false confidence |
|---|---|---|
| Chronological/following feed | Correct data mode across pages and after restart | Only the label changed; one page was sorted locally |
| Manual pagination | Stops at the end; only a tap adds data; the next boundary stops again | No network; endless spinner; automatic request already sent |
| Video swipe restriction | Same item after fast/slow gestures in both directions | Gestures sent on home; initial loading unfinished; downward refresh replaces video |
| Auto-next restriction | No transition at the video's actual end | Waiting briefly on a long video |
| Tab redirection | Intended package is foreground; missing target handled clearly | A log only states an intent to launch |
| Push filtering | A blocked incoming event produces no display; an allowed category works | No incoming event occurred |
| Button styling | Accessible label, adequate hit area, theme, correct callback | A good screenshot without a working tap |

Run scenarios relevant to the change. After a purely visual button update, inspect presentation, accessibility/click behavior, and evidence that logic is unchanged; a full app audit need not be repeated.

Keep tokens, passwords, and private messages out of logs. Useful events include manual page requested, auto-page blocked, viewer gesture canceled, and target launched. Match intention logs to UI/request state to establish the actual outcome.

## Result record

Record:

- Input/output fingerprints, package, and OS/device version.
- Changed surfaces and retained functions.
- Change-point table with semantic checks.
- Static checks, runtime scenarios, and observed results.
- Unverified variants/experiments and rollback approach.
- Output artifact path and whether it was installed.

Do not transfer a test result between versions without comparing the relevant code. “78 seconds without a transition” is a bounded observation, not proof of auto-next behavior for every video.
