---
name: instagram-452-1-0-53-64
description: "Version-specific unrotapps subskill for Android Instagram 452.1.0.53.64 arm64, versionCode 385811006: manual home/Explore pagination, Reels restrictions, and native button styling. Reinvestigate before using this symbol map on another build."
---

# Instagram 452.1.0.53.64 / arm64

The main [unrotapps skill](../../SKILL.md) loads this file for Instagram work. This map supplements the general rules and is not an automatic installer. If invoked directly, first read the main SKILL.md and relevant shared references.

## Fingerprint and evidence

| Field | Investigated value |
|---|---|
| Original package | com.instagram.android |
| versionName / versionCode | 452.1.0.53.64 / 385811006 |
| ABI, DEX | arm64, 23 DEX |
| Base APK SHA-256 | 22f20062d16997e26009e73b0425974efef8ac69c76dd8912ee8a091213c39e2 |
| Input APKS SHA-256 | d992ab78fa5f20d0032afef85973988e3380a67196d691adae6af19084298d85 |
| Investigated density splits | ldpi, mdpi, hdpi, xhdpi, xxhdpi, xxxhdpi |
| Device validation | Pixel 10 Pro, Android 17; 2026-10-25 |

Original base/density split signatures were verified against the published [Instagram certificate pins](https://www.instagram.com/.well-known/assetlinks.json). Recheck signatures and a current trusted reference for each new use. This supports input provenance, not a guarantee against harmful behavior.

A matching versionName alone is insufficient for automatic reuse. If only the container differs, compare base, splits, and relevant classes. If the base differs, resolve all symbols and resource IDs again.

Task parameters: CLONE_PACKAGE, DISPLAY_NAME, MODE, and selected surfaces. Advanced mode may additionally require ACTION (disable / external app / reminder), TARGET_PACKAGE, or REMINDER_TEXT. Personal values are not part of this skill.

## Applying the modes

**Light:** retain original navigation and buttons. Apply only restrictions on continuous paging in the chosen viewer and/or manual pagination in the chosen feed. Skip the Reels-entry redirection below. Opening a specific video through retained entry points must work.

**Advanced:** disable selected buttons or assign the user's chosen action. If full feature blocking is requested, investigate cards, profiles, messages, deep links, and other entries too: the tab map below does not cover them all. Reminders require a separate local screen/dialog; no tested reminder patch is included. Successful tab reassignment does not establish complete Reels blocking.

Neither mode automatically includes chronological feeds or push filtering. The table records evidence for individual mechanisms, not complete tested builds of both modes.

## Implemented and verified scope

| Mechanism | Status |
|---|---|
| Home feed → manual pagination | Eager-prefetch/direct-loader bypasses fixed; 50 swipes across two pages and one manual load tested |
| Explore grid → manual pagination | Implemented and tested separately |
| Reels tab → chosen external app | Implemented; foreground target package checked |
| Vertical Reels swipes, including downward refresh | Implemented; fast/slow gestures tested in both directions |
| Auto-next | Known methods disabled; 78 seconds without transition observed, not a test of every video duration |
| Native-style button | IgdsButton appearance and tap checked in the current dark theme |
| Forced chronological / friends-only / ad-free feed | Not implemented; investigation leads below |
| Push filtering | Not investigated or implemented in this case |
| Profile/search/message/comment pagination | Unchanged |

## Verified change points

Search by `.class`. The `.1` suffix below comes from decoding on macOS and is not part of the class name.

### Home: classes2.dex

**LX/00tU;** (decoded as X/00tU.1.smali) implements LX/0pgh;:

- A00(Map): automatic loading path; inspect its call sites.
- A01(Map;Z): next-page request.
- EiF(): existing load/retry entry point, also called internally.
- A02(): actual busy/loading state.
- EER(): nonempty cursor available.
- EVZ(): footer eligibility.
- isLoading(): original logic includes waiting for an automatic next page; leaving it intact can produce a permanent spinner.

Issue a separate one-shot boolean only in the explicit click handler. `manualPagePermit` is an illustrative name for an added field, not an original symbol. EiF alone does not establish user intent. A dedicated manual entry point calls EiF and clears the permit on early return or exception. EiF checks A02; A01 consumes the permit. Use A02 for isLoading and EVZ for footer availability. Preserve original initial-load and refresh paths.

Additional bypasses found while investigating rapid scrolling:

- **LX/01Am;.GrS:** shared loader; LX/02ov;.A0Q means PAGINATION. Direct calls, including `eager_head_load_active_group`, bypass the policy gate. Admit PAGINATION only with a one-shot marker from manual A01; remove it before constructing the network request. Do not block other enums by resemblance: A0N means NAVIGATE_OFF_SURFACE_EAGER_REFRESH.
- **LX/03mQ;.A00:** calls LX/0Neg;.AVU independently of policy A00. Its concrete **LX/02pQ;.AVU** implementation prefetches the home feed through LX/0Jkm;.HcI or LX/0Jeo;.GrU, bypassing GrS. Disable this prefetch for manual pagination while retaining the regular manual request. Do not disable methods named AVU globally.

This is a fix map. A successful build/install still needs tests across multiple page boundaries with rapid gestures and explicit taps.

Check other A01 call sites and direct LX/00qR;.A0K/A0J calls. One listener cannot be assumed to cover every scenario. `empty_head_load_tail_retry` has different semantics from ordinary scroll prefetch.

#### Home regression evidence

The investigated build was tested after the fix:

1. 24 rapid 60 ms swipes reached the Load more footer.
2. Another 8 swipes lasting 30–700 ms left the footer in place; no manual request was admitted without a tap.
3. One tap admitted exactly one next-page request; a new post replaced the footer.
4. Another 18 rapid 50 ms swipes reached the next footer. The admitted manual-request count remained one.
5. Runtime logs confirmed the separate eager-prefetch path was invoked and blocked. Installed/local APK SHA-256 matched, and the existing signed-in session was retained.

For reproduction, derive coordinates from the current UI tree, allow momentum to settle, and confirm the home feed is open. Count admitted requests, not just gestures or intent logs. Already loaded posts remain scrollable. This does not cover every server experiment or transfer to Explore; test rapid Explore gestures separately.

### Explore: classes5.dex

**LX/093k;** has the original-name marker ExploreFragment.

- A0H: LX/0DFN; store/loader.
- LX/0Dnj;: LX/0pgh; implementation, manual EiF, and footer states.
- LX/0HAb; and LX/0Dlr;: alternative request controllers.
- LX/0DFN;.A0C(LX/0DGk;): shared point in the investigated path.
- LX/0DFN;.A0L: current cursor.
- LX/0DGk;.A0E: flag allowing initial load/refresh in this chain; establish its semantics from constructors and call sites.

Implemented gate: after isLoading checks in EiF, issue a one-shot permit on the corresponding DFN. In A0C, require that permit for a nonempty cursor when A0E is absent; consume it on admission. This preserves initial loading and refresh while stopping ordinary automatic pagination. Check alternative request paths for the current account.

### Footer and styling: classes3.dex

- com/instagram/ui/widget/loadmore/LoadMoreButton;.setViewType(LX/0pgh;LX/0EAk;)
- LX/01Vd;: footer state selection.
- Added helper: click listener and scoped bind; choose its name during implementation, as it is not an original APK class.

For `LX/00tU;`, the click listener must call the dedicated manual entry point that issues the permit. A direct EiF call no longer issues permission and will not load a page. Retain the investigated Explore path for `LX/0Dnj;`; do not transfer the new home gate without analysis. The added method's name is an implementation choice, not an original Instagram symbol.

Bind only for LX/00tU; and LX/0Dnj; policies when a next page exists and there is no loading/error. Preserve the existing retry behavior. This layout uses child index 2 for the manual footer; inspect ViewAnimator structure before reusing the index.

Styling uses **com.instagram.igds.components.button.IgdsButton** (classes4):

- constructor(Context), setText(CharSequence), View.setOnClickListener.
- setStyle(LX/04Tw;.A04): PRIMARY.
- setSize(LX/04UB;.A07): LARGE.
- FrameLayout.LayoutParams: MATCH_PARENT × WRAP_CONTENT, centered, with 16 dp side margins converted using DisplayMetrics.density.

IgdsButton extends FrameLayout, not TextView. Remove incompatible Button/TextView.setText or setAllCaps calls when changing types. The native component follows the current theme; only the dark theme was visually verified in this case.

### Reels entry: classes2.dex

- LX/08kM;.A08: CLIPS.
- LX/00JF;: onClick/onTouch, enum in A02, Activity in A00.
- LX/00JM;: additional onClick, enum in A02, Activity in A01.

For advanced-mode external-app reassignment only: when CLIPS matches, launch TARGET_PACKAGE before original navigation. onTouch should suppress Reels preloading while retaining click delivery. Add a narrow manifest package query for the chosen target if needed. Handle ActivityNotFound/Security with a clear message, without falling back into Reels. Match launch logs to the foreground Activity.

For a disabled button without a replacement, check both touch and accessibility click for navigation/preloading. For a reminder, use REMINDER_TEXT and explicit dismissal; implement and test that variant separately.

### Reels viewer: classes5.dex and classes2.dex

**LX/04QD;**, with ViewPager2 in A0H:

- HRh(View): disable user input immediately after assigning the pager.
- B9G(): disable input; BCv(): prevent re-enabling it.
- GwQ(Z), GwV(), HXd(), GwE(F): investigated next/previous/smooth/peek transitions, disabled.
- Preserve initial H3q/H3r/setCurrentItem behavior unless investigation establishes how opening a specific media item works.

**LX/04QU;**:

- A03(LX/04QU;Z): early return for auto-next.
- A06(LX/04QU;): false for should-scroll-next.

Later controllers may re-enable input. In **androidx.viewpager2.widget.ViewPager2**, a guard scoped to getId() == **0x7f0b0cbc / clips_viewer_view_pager** keeps input disabled. Other ViewPager2 instances must retain their behavior.

Additional dispatchTouchEvent protection for that same ID:

- DOWN stores X/Y and resets the gesture flag.
- Predominantly vertical movement above the threshold cancels the child gesture with ACTION_CANCEL.
- Restore the original action; consume the rest of the gesture.
- Preserve ordinary taps and horizontal motion.

The case used a 24 px threshold. It is a solution parameter, not a universal touch-slop recommendation.

**Separate critical path:** parent `instagram/features/clips/viewer/ui/ClipsSwipeRefreshLayout` (classes5), ID **0x7f0b0c67 / clips_swipe_refresh_container**.

A long downward swipe triggered refresh and replaced the video despite working upward-swipe blocking. For this ID:

- onInterceptTouchEvent/onTouchEvent return false.
- dispatchTouchEvent delegates directly to ViewGroup, bypassing the component's refresh handling.

Preserve other IDs. This finding does not justify disabling home-feed refresh.

## Chronological feed: investigation leads only

These are not a completed patch:

- LX/02sV;: FOLLOWING = A0E, FAVORITES = A0D, RECENTS = A0G; BLENDED/BLENDED_FOR_YOU also occur.
- LX/00qF;: last_selected_feed_type; reads/writes depend on MobileConfig 0x810e4c00064e2f.
- LX/02tL;: mode selection and conditional persistence.
- LX/02pl;: request builder using pagination_source.

Verify the actual request and subsequent pages rather than merely forcing a flag. RECENTS has ambiguous server semantics; one enum does not establish a friends-only, chronological, ad-free feed.

For push, start with the [general methods](../../references/mechanisms.md). No Instagram-specific classes/channels have been established here.

## Build and client isolation

Apply the [Android invariants](../../references/android-apk.md). Additional findings for this build:

- After each DEX edit, update the corresponding SHA-1 in assets/secondary-program-dex-jars/metadata.txt.
- Package renaming used an exact string/authority/permission mapping and correct DEX reserialization.
- The resource package name in resources.arsc was changed while preserving numeric IDs.
- Application label retains resource reference **0x7f130009**; launcher aliases may have their own visible label.
- InstagramAuthenticationService/AccountAuthenticator registration was investigated and disabled in the clone for account isolation. Reassess this for a new implementation.
- The official i18n_ru_RU module was obtained from an installed copy of the **same version**, its signature verified, and its payload preserved while adapting package/signature to the clone.
- LX/0AC8; acquired merged try ranges after assembly. Equivalence was established by comparing instructions and the first applicable handler at every instruction. This is not permission to ignore arbitrary changes to that class.

The rapid-scroll iteration changed only LX/00tU;, LX/01Am;, LX/02pQ; logically in classes2.dex and the added footer helper in classes3.dex relative to the previous verified build. Other classes matched canonical disassembly; other DEX remained byte-identical. Loader hashes, CRC, DEX checksums, signatures, and alignment were checked. Reels logic was unchanged in that iteration. This describes one verified delta, not an allowlist for other builds.

Signing keys, passwords, APKs, private logs, and account data are excluded from this subskill. When an existing clone is present, locate its previous key/artifacts within the authorized workspace before updating; do not silently generate an incompatible key.

## Reproducibility

This map is not an automatic patcher. For new work, retain fingerprints, changed-symbol mappings, diffs, build logs, and device-test evidence in the workspace. Keep secrets and personal parameter values out of distributed instructions/comments. If earlier artifacts are unavailable, investigate the available APK again using this map rather than assuming the previous result is reproduced.
