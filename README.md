# unrotapps

[Русский](README-ru.md)

**Doomscrolling isn't your fault.**

You open an app to answer a message, look at a friend's photo, or watch the video someone sent you. Twenty minutes later you're still there. The feed never ends, the next video starts on its own, and every time you think about closing the app, there's already something new on screen.

That's not weak willpower. Infinite feeds, autoplay and swipe-to-next-video are built by teams whose job is to keep you in the app a little longer. They're very good at it.

And you probably don't want to delete the app. Your friends are there, your chats are there, some of what you see is genuinely worth seeing. What's missing is an ending: a point where the feed stops and you decide whether to keep going.

unrotapps is an attempt to put that ending back. You tell an AI coding assistant what bothers you, and it changes that specific thing in the app: the feed stops after a page and shows a **Load more** button, Reels no longer swipe to the next one, a tab you open on autopilot stops working. Everything else stays as it was.

## What it is

unrotapps is a skill for AI coding assistants: it works in Claude Code and Codex. It contains instructions, investigation methods and notes on specific app versions. You describe the behavior you want. The assistant first checks whether the app already has a setting for it. If not, it digs into the app, makes the smallest change that works, and tests the result.

| You want | What the assistant can look into |
|---|---|
| A feed that ends | Replace automatic loading with a **Load more** button, including hidden preloading |
| To watch just the video you opened | Keep the video, block the swipe, auto-next and refresh that replace it |
| Fewer habitual taps | Disable chosen buttons, or point them to another app or your own reminder |
| Control over what you see | Find chronological, following or favorites modes, if the app has them, and check what they really show |
| Fewer interruptions | Turn off some notification types and keep the ones you need |

Not everything is possible in every app. It depends on the app, its version, the platform and what the server decides. If the app already has a setting that does the job, the assistant uses it.

## Two modes

### Light: keep everything, stop the endless scroll

All features and buttons stay. Only the way you continue changes:

- You scroll through what's loaded, then tap **Load more** to get the next page.
- You can open a specific video, but you can't swipe into the next one.
- Buttons go where they always went.

### Advanced: remove the entry points

The buttons for a feature you want to avoid stop working. If you like, they can open another app instead, or show a reminder you wrote.

To block a feature completely, the assistant also checks its other entry points. A hidden tab can still be reachable through links, cards or messages.

Feed order, who shows up in the feed, ads and notifications are separate requests. Neither mode touches them unless you ask.

## Getting started

Copy the whole `unrotapps` folder, with its `references` and `subskills` folders, into your assistant's skills directory:

| Assistant | Where to put it | How to call it |
|---|---|---|
| Claude Code | `~/.claude/skills/unrotapps/` (or `.claude/skills/` inside a project) | `/unrotapps`, or just describe the task and Claude picks the skill up |
| Codex | `~/.codex/skills/unrotapps/` | `$unrotapps` |

The `agents` folder holds Codex display settings; Claude Code ignores it. If you use both assistants, keep one copy and symlink it:

```bash
ln -s ~/.codex/skills/unrotapps ~/.claude/skills/unrotapps
```

Then describe the app and what you want changed. Any language works.

Examples:

```text
/unrotapps Light mode for Instagram on Android.
Keep messages and profiles working. Make the home feed load the next
page only when I tap Load more, and stop Reels from swiping to the next one.
```

```text
/unrotapps Advanced mode. Disable the short-video tab and show
my reminder when I tap it: "What did I come here to do?"
Check the other ways into that section too. Keep messages working.
```

```text
/unrotapps Check whether this app can show posts from accounts
I follow in chronological order. Keep direct-message notifications
and turn off recommendation notifications if possible.
```

In Codex, start with `$unrotapps` instead of `/unrotapps`.

If you know the app version, mention it, and name the screens you care about. If you don't, the assistant will help you find the version and get the files it needs.

## Android

Android has the most detailed workflow in this skill. You'll need a computer that can run the tools, a USB cable that carries data (not just charging), and a phone with **USB debugging** turned on. Unlock the phone and confirm the debugging prompt on it.

You probably won't need to download an APK. The assistant can usually copy the installed app straight from your phone, including the extra files known as *split APKs*. An official copy from your phone or from the publisher is best. The skill checks where a package came from and whether its signatures match.

If the change requires re-signing the app, you'll usually end up with a separate clone next to the official app. You may need to log in again in the clone, and some apps refuse to work with modified clients. To update the clone later, you'll need the same signing key. The skill leaves the original app in place and doesn't wipe app data to get around an install error.

Passwords, PINs and fingerprint confirmations stay on your phone. Step-by-step instructions: [Android setup, extraction and installation](references/android-device.md).

## What's been tested

The included [Instagram subskill](subskills/instagram-452-1-0-53-64/SKILL.md) is for **Android Instagram 452.1.0.53.64, arm64**, and checks that the input file matches that exact build.

Tested on a real device: manual pagination in the home feed and Explore, blocked swipes in Reels, a tab reassigned to another app, and a **Load more** button made from Instagram's own UI components. The home feed fix held up through **50 swipes across two pages**, fast swipes included, with a manual load in between.

Not done in the Instagram subskill: chronological or friends-only feed, ad removal, notification filtering. There are leads for feed modes, but no working change. The reminder option needs its own implementation and testing. Auto-next was checked only briefly and not across all playback settings.

Other Instagram versions need a fresh investigation. The general methods help with that, but the skill doesn't ship a ready-made modified APK or a patcher that works on any version.

## What you get at the end

The assistant should tell you plainly what it changed, what it tested on the device, what it only confirmed by reading the code, and what is still unknown. If it built an installable app, it should tell you where the file is and whether it was installed.

An app update may undo the changes. Signature checks show where a file came from, but they're not a full malware audit. Some behavior is controlled by the server and can't be changed on your phone.

The goal is simple: you open the app for something you need, reach the end, and decide for yourself whether to keep going.
