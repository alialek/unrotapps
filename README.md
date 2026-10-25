# unrotapps

[Русский](README-ru.md)

**Give infinite feeds an end. Go touch grass.**

![Claude Code](https://img.shields.io/badge/Claude_Code-skill-D97757)
![Codex](https://img.shields.io/badge/Codex-skill-111111)
![Tested](https://img.shields.io/badge/tested-Instagram_452_on_Android-E4405F)

<p align="center">
  <img src="assets/load-more.png" width="360" alt="Instagram home feed ending with a Load more button">
  <br>
  <sub>Instagram home feed after unrotapps: one page, then you decide.</sub>
</p>

You open Instagram to reply to one message. Twenty minutes later you're still scrolling. That's not weak willpower: infinite feeds and swipe-to-next-video are built by teams whose job is to keep you in the app. They're good at it.

You probably don't want to delete the app. Your friends and chats are there. What's missing is an ending. unrotapps lets your AI coding assistant remove the parts that keep you hooked and leave everything else alone.

## Before and after

| | Before | After |
|---|---|---|
| Home feed | Never ends | One page, then a **Load more** button |
| Reels | Swipe, next video, swipe, next video | You watch the one you opened |
| Reels tab | Opens on autopilot | Opens another app instead |
| Feed content | Picked by the algorithm | Following or chronological mode, if the app has one |
| Notifications | Everything | Only the types you keep |

The first three rows are tested on Instagram for Android. The last two are things the assistant can investigate, with no tested change yet (details under *What's been tested* below).

## Quick start

1. **Install.** Put the `unrotapps` folder into your assistant's skills directory:
   - Claude Code: `~/.claude/skills/unrotapps/`
   - Codex: `~/.codex/skills/unrotapps/`

   Using both? Keep one copy and symlink it:

   ```bash
   ln -s ~/.codex/skills/unrotapps ~/.claude/skills/unrotapps
   ```

2. **Call it.** Type `/unrotapps` in Claude Code or `$unrotapps` in Codex. In Claude Code you can also just describe the task.

3. **Say what bothers you.** In any language:

   ```text
   /unrotapps Light mode, Instagram on Android. Keep messages and profiles.
   Load the next home feed page only when I tap Load more,
   and stop Reels from swiping to the next one.
   ```

The assistant checks the app's own settings first. If they don't help, it digs into the app, makes the smallest change that works, and tests it on your phone.

## Two modes

**Light** keeps every feature and button. You scroll through what's loaded and tap **Load more** for the next page. You can open a specific video but can't swipe into the next one.

**Advanced** turns off the entry points to a feature you want gone. Buttons can do nothing, open another app, or show a reminder you wrote. For full blocking, the assistant also checks other ways in: links, cards, messages.

Feed order, ads and notifications are separate requests. Neither mode touches them unless you ask.

<details>
<summary><b>More examples</b></summary>

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

If you know the app version, mention it, and name the screens you care about. If you don't, the assistant will help you find out.

</details>

## FAQ

**Will I get banned?**
No guarantees. Modified clients go against Instagram's terms, and some apps detect them. The modified app is installed as a separate clone with its own login, but the account is the same. Your call.

**Do I need root?**
No. USB debugging is enough. The skill tells the assistant not to root the phone or unlock the bootloader.

**iPhone?**
The detailed workflow is Android only. On iOS, web and desktop the assistant can only use settings and officially supported ways to extend an app.

**What happens when the app updates?**
The clone is a separate app, so store updates won't touch it, and it won't get new versions on its own either. A new app version needs a fresh investigation; the existing notes help but don't apply blindly.

**Is my original app safe?**
The original stays installed. App data isn't wiped to get around install errors. Passwords, PINs and fingerprints are entered only by you, on the phone.

<details>
<summary><b>Android setup</b></summary>

You'll need a computer that can run the tools, a USB cable that carries data (not just charging), and a phone with **USB debugging** turned on. Unlock the phone and confirm the debugging prompt on it.

You probably won't need to download an APK. The assistant can usually copy the installed app straight from your phone, including the extra files known as *split APKs*. An official copy from your phone or from the publisher is best. The skill checks where a package came from and whether its signatures match.

If the change requires re-signing the app, you'll end up with a separate clone next to the official app. You may need to log in again in the clone, and some apps refuse to work with modified clients. To update the clone later, you'll need the same signing key.

Step-by-step instructions: [Android setup, extraction and installation](references/android-device.md).

</details>

<details>
<summary><b>What's been tested</b></summary>

The included [Instagram subskill](subskills/instagram-452-1-0-53-64/SKILL.md) is for **Android Instagram 452.1.0.53.64, arm64**, and checks that the input file matches that exact build.

Tested on a real device: manual pagination in the home feed and Explore, blocked swipes in Reels, a tab reassigned to another app, and a **Load more** button made from Instagram's own UI components. The home feed fix held up through **50 swipes across two pages**, fast swipes included, with a manual load in between.

Not done in the Instagram subskill: chronological or friends-only feed, ad removal, notification filtering. There are leads for feed modes, but no working change. The reminder option needs its own implementation and testing. Auto-next was checked only briefly and not across all playback settings.

The skill doesn't ship a ready-made modified APK or a patcher that works on any version.

</details>

<details>
<summary><b>What you get at the end of a session</b></summary>

The assistant should tell you plainly what it changed, what it tested on the device, what it only confirmed by reading the code, and what is still unknown. If it built an installable app, it should tell you where the file is and whether it was installed.

Signature checks show where a file came from, but they're not a full malware audit. Some behavior is controlled by the server and can't be changed on your phone.

</details>

---

The goal is simple: open the app for something you need, reach the end, and decide for yourself whether to keep going.
