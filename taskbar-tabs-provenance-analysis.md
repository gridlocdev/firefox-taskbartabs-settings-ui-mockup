# Manually-added tabs vs. installed web apps: a design analysis

The current design treats every entry in the list as the same kind of object: a name and an address the user chose. That is true for a Taskbar Tab someone added from the address bar. It is not true for a site that ships a `.webmanifest`, where the developer declares the app's name, icon, start page and scope, and those values are the app's identity rather than the user's preference.

This document works out how the interface should represent that difference, and audits the result against Jakob Nielsen's ten usability heuristics.

**Premise:** Firefox's Taskbar Tabs today derives a name and icon from the page, not from a manifest, so the second kind of entry does not exist yet. This is deliberately ahead of that. The interface for developer-owned identity is the part that has to be settled before the first app arrives carrying one, not after, and Chrome's experience, covered in §5, is what a browser retrofitting it looks like.

**Status:** built. The prototype in [mockup.html](mockup.html) implements all of it, and the [specification](taskbar-tabs-options-specification.md) carries the behaviour: §3.1 for the badge, §4.1 and §4.2 for the editing rules, §4.11 and §4.12 for the two information bars, §5.1 for the add flow, and §9 for the summary.

---

## 1. What actually differs

Provenance is not one difference. It is a set of them, and only some justify a change in the interface.

| | Added by the user | Declared in a manifest |
|---|---|---|
| **Name** | User typed it, or it was derived from the host | `name` / `short_name`, developer-owned, **can change when the manifest is re-fetched** |
| **Address it opens** | Whatever URL the user gave | `start_url` |
| **Area it covers** | Inferred: the host of the address | `scope`, declared explicitly; defaults to the directory of `start_url` |
| **Identity across changes** | The row itself | `id`, which is how you tell "same app, renamed" from "different app" |
| **Icon** | Favicon, or a letter tile | `icons`, sized and purpose-tagged |
| **Window shape** | Firefox's choice | `display` (`standalone`, `minimal-ui`, …) |
| **What removal means** | Delete a shortcut | Delete a shortcut. Eventually also: handler registrations, badging, notification state |

Two of these matter enough to change the interface. The rest are implementation.

**The name can change without the user doing anything.** No other value on this page behaves that way. An app the user pinned as "Outlook" can become "Microsoft Outlook" because a developer edited a JSON file. The interface has to have an answer for that.

**The address is an identity claim, not a preference.** A window that presents itself as an app, carries the site's icon, sits on the taskbar and holds the site's cookies and permissions is making a statement about what it is. Letting the address be repointed anywhere turns that into a spoofing primitive: an icon labelled Outlook, wearing Outlook's icon, opening somewhere else. The reason to constrain it is not that the developer said so. It is that the identity has to stay true.

Everything else, meaning icon, window shape and removal mechanics, is invisible to the person using the page, and should stay that way.

---

## 2. The recommendation in one table

| Field | Added by the user | From a manifest |
|---|---|---|
| **Name** | Free text | **Free text**, with the site-provided name shown permanently beneath it and a **Use the name from the site** reset |
| **Address** | Any valid `http(s)` URL | **Editable, constrained to the app's scope.** Defaults to `start_url`. Out-of-scope input is rejected with an explanation and a way out |
| **Icon** | Not editable | Not editable |
| **Everything else** (container, login, link capture, permissions, data, removal) | Unchanged | Unchanged |

This departs from the instinct to lock the name. The argument is in §4.

---

## 3. Representing it in the list

### One list, not two

The obvious move is to split the list into "Installed apps" and "Shortcuts". It should be resisted.

Someone looking for Teams does not know, and should not have to know, whether Teams arrived through a manifest. Splitting forces them to recall provenance in order to locate the thing, which is the exact failure heuristic 6 exists to prevent. It also doubles the empty states, breaks "Select all", and makes sort order mean two different things at once. The distinction is real, but it is a property of a row, not an organising principle for the page.

Filters and a Type column were also considered and rejected. Six apps do not need a filter, and a column spends permanent horizontal space on a fact that matters on perhaps two screens.

### A badge, matching the badges that already exist

The row already carries a set of read-only labels for pinned, container, opens at login and exact page, and provenance is the same kind of fact. It joins them.

```
┌─────────────────────────────────────────────────────────────────┐
│ ☐  [O]  Outlook                                    [Open]  [⋯]  │
│         https://outlook.office.com/mail/  ·  Today at 09:12     │
│         From the site   Pinned to taskbar   ●Work   Exact page  │
└─────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│ ☐  [H]  Home Assistant                             [Open]  [⋯]  │
│         http://homeassistant.local:8123/lovelace/0  ·  07:55    │
│         Added by you   Pinned to taskbar   ●Personal   Exact…   │
└─────────────────────────────────────────────────────────────────┘
```

Both states get a badge rather than only one, following the precedent already set by **Pinned to taskbar** / **Not pinned**. Absence of a badge is a weak signal; a reader cannot distinguish "this app has no badge" from "I did not notice the badge."

Manifest icons are full-colour and designed, where manual entries fall back to a letter tile, so in practice the two kinds will look different before anyone reads a word. That is a useful pre-attentive cue and it costs nothing, but it cannot be the only one, since the design already commits to never making colour the sole signal.

### Wording

**Recommended: "From the site" and "Added by you".**

The runner-up was "Installed app" and "Shortcut", borrowed from Chromium and from mobile. It has real recognition value and it lost anyway, because both kinds of entry install identically, the same shortcut file, written the same way. Calling one of them "installed" asserts a mechanical difference that does not exist, which is the same class of fiction this design already rejected when it removed the unpin toggle.

"From the site" and "Added by you" describe provenance exactly, and they explain the editing rules directly: the site defined it, so the site owns the name.

**The badge must not imply endorsement.** No checkmark, no shield, no "Verified". A manifest is asserted by whoever controls the origin and Firefox verifies nothing beyond that. A trust-shaped badge would be a security claim the browser cannot back.

---

## 4. Representing it on the per-app page

### The pattern already exists on this page

Site permissions (§4.8) are shown and not edited, with a sentence explaining that the truth lives at site scope and a link to where it can be changed honestly. Manifest-owned fields are the same shape of problem, since the truth lives with the site, and should reuse that pattern rather than invent one.

This is the strongest structural argument in the analysis. Provenance does not need a new visual language. It needs the one the page already uses for "here is the real state, and here is where it actually lives."

### One sentence at the top

Under the app header, for manifest apps only:

> Outlook provides its own name, icon and start page. You can rename it here for yourself.

Not a card, not a banner, not a dialog. One line of muted body text, placed where a reader hits it before they reach the first field they cannot freely change.

### Name

```
Name
┌───────────────────────────────────────────────────────────┐
│ Outlook (Work)                                            │
└───────────────────────────────────────────────────────────┘
Shown in the Start menu and Alt+Tab. The site calls this app
"Microsoft Outlook".  [Use the name from the site]
```

The field stays editable. The site's name sits underneath it permanently, not only while overridden, so the developer's claim is always legible. The reset link appears only when the two differ.

**Why not lock it.** The name is a private label. It is never transmitted, never used in an origin decision, never a security boundary. Meanwhile the single most-requested thing missing from Chromium's version of this feature is renaming an installed app, because installing the same site across several profiles leaves you with several identically named icons, which is precisely the manifest case. Locking the name would create a dead end for the only scenario where renaming is genuinely necessary, in exchange for no security gain, since the spoofing risk lives entirely in the address. The developer's claim is preserved by showing the provided name, not by suppressing the user's.

### Address

```
Address
┌───────────────────────────────────────────┐  ┌──────────────────┐
│ https://outlook.office.com/mail/inbox     │  │ Use start page   │
└───────────────────────────────────────────┘  └──────────────────┘
Must stay within outlook.office.com/mail/, the area this app covers.
```

The field accepts input and validates it, rather than sitting disabled. In-scope pages are allowed, so "open Outlook at my calendar instead of my inbox" works. Out-of-scope pages are rejected:

> `example.com` is outside this app's area. Outlook covers `outlook.office.com/mail/`. To open a different site in its own window, **add a Taskbar Tab** instead.

That message diagnoses the problem, states the rule, and hands over a working alternative in one breath. The link prefills the Add dialog with what was typed.

**Why constrained rather than locked.** A disabled field answers "you can't" and nothing else, and the user is left to guess whether it is broken, unsupported or forbidden. A validated field teaches the rule at the moment it matters, which is what heuristic 5 actually asks for, and it happens to be exactly what the manifest specifies, since `scope` is the app's declared area, so honouring it is not a restriction Firefox invented.

For **Use home page** (§4.3): on a manifest app the button becomes **Use start page** and resets to `start_url`, which is the developer's answer to the same question the button was asking.

### No advanced override

An escape hatch for out-of-scope addresses should not be built, hidden or otherwise.

Every legitimate need is already met: in-scope pages are editable, and any other URL can be added as its own Taskbar Tab in two clicks. What an override would add is exclusively the illegitimate case: an entry that keeps an app's name and icon while pointing somewhere else. Hidden advanced flows are also, in practice, worst for the users most likely to be talked into using them; "open settings, expand Advanced, paste this" is a support-scam script.

The escape hatch is a different object, honestly labelled, not a weakened version of this one.

---

## 5. Flows that change

### Adding

The Add dialog validates the address already. When the address resolves to a site with a manifest, the dialog offers the choice rather than deciding silently:

```
Address   https://outlook.office.com                          ✓

  ◉  Add Outlook, the app this site provides
     Uses the name, icon and start page from outlook.office.com

  ○  Add a shortcut to this page
     You choose the name and the exact address

Name      Outlook
Container No container            ▾
☑ Pin to taskbar
```

Choosing the app fills the name from the manifest and locks the address to `start_url`; choosing the shortcut leaves both free. The address-bar button follows the same logic: **Install Outlook** where a manifest exists, **Add to taskbar** where one does not.

### A site starts offering an app

A manual shortcut to a site that later ships a manifest gets one row on its settings page:

> **This site provides an app.** Outlook offers its own name, icon and start page.
> **Switch to the app…**

Switching keeps the container, login and link-capture settings and adopts the manifest identity. It rebuilds the shortcut, so the existing re-pin confirmation (§5.2) applies unchanged.

### The developer renames the app

The case with no current answer, and the one most likely to be discovered in the field rather than in review.

| Situation | Behaviour |
|---|---|
| User has set their own name | Keep it. Never overwrite a local name. The "site calls this app…" line updates to the new value |
| User has not | Adopt the new name and rebuild the shortcut |

Adoption is not silent. The settings page shows an information bar until dismissed:

> outlook.office.com renamed this app from **Outlook** to **Microsoft Outlook** on July 12.
> [Keep calling it Outlook]  [Dismiss]

The date carries its year only when it is not the current one: **July 12**, or **July 12, 2025**.

"Keep calling it Outlook" simply writes a local override, which is the same mechanism as a manual rename, so no new state is introduced. The manifest `id` is what makes this a rename rather than a new app.

**Chrome built the same mechanism**, which is worth naming because it is convergent evidence rather than a borrowed idea. Their framing of the problem: *"Because Web Apps lack a central authority like Google Play to review updates, these modifications must be presented clearly to users for confirmation."* [Chrome 144](https://developer.chrome.com/blog/improvements-to-web-app-updates) splits the manifest so that icons and names are *"security sensitive members"* held separately, everything else applies immediately, and an identity change waits behind a **Review app update** entry on the app's own menu. The version before it forced *"a disruptive dialog requiring them to either uninstall the app or immediately accept icon and name changes"*, the modal this design also rejects.

The one difference is that Chrome holds the change pending and asks for acceptance, where this adopts it and offers the decline. The local override is what makes that affordable: declining is not blocking an update, it is writing a label, so nothing is left holding a stale identity to reconcile. Chrome's other rules transfer unchanged. Sub-10% icon differences applying automatically belongs with an icon pipeline this design does not have, and `start_url` and `scope` changes applying as configuration is what keeps the address constraint moving with the app rather than pinned to install day. The rationale carries the longer version.

### Removal

Removal stays **Remove Taskbar Tab** for both kinds, with the same confirmation. What is being removed is the Taskbar Tab; the manifest is not uninstalled and the site remains available in a normal tab, which is exactly what the existing dialog says. "Uninstall" would import an expectation of more thorough removal than actually happens.

Revisit this if manifest apps ever gain real installed state: handler registrations, a separate storage bucket, OS-level registration. Then "Uninstall" would be the honest word and the confirmation would need to enumerate what goes.

### Search

Search matches the site-provided name as well as the local one. Someone who renamed an app to "Work Mail" may well search for "Outlook", and being told there are no results for an app sitting in the list is a bad failure.

### Duplicates

A manual shortcut and a manifest app for the same start URL should not both be creatable without comment. The Add dialog should say so and offer to open the existing one.

---

## 6. Heuristic audit

### 1. Visibility of system status

A row states which kind it is. A constrained field states its constraint before it is violated, not after. A name changed by a developer is announced rather than discovered.

The failure avoided is the silent one: an app whose name changed overnight, or a field that will not accept input for reasons the page never mentions.

### 2. Match between the system and the real world

No `manifest`, `scope`, `start_url` or `id` appears in the interface. "The area this app covers", "the start page", "the name from the site". The scope rule is expressed as the URL prefix a reader can compare against what they typed, not as a specification concept.

"Added by you" and "From the site" are statements about where something came from, which is a question people can answer about their own computer.

### 3. User control and freedom

The name is overridable and resettable. The start page is adjustable within the app's own area. A URL outside it is reachable as its own Taskbar Tab. A developer's rename can be declined.

No branch of this design ends in "you cannot, and there is nothing else to try."

### 4. Consistency and standards

Internally: provenance reuses the read-only-with-explanation pattern from Site permissions, and the both-states badge pattern from Pinned / Not pinned. Externally: the install-versus-shortcut split matches Chromium, and manifest fields are honoured as specified rather than reinterpreted.

The violation avoided is the sharpest one available here, namely two rows that look identical and behave differently when clicked.

### 5. Error prevention

The address is validated against scope rather than disabled, so the rule is learned at the point of use. The class of error this really prevents is not a typo: it is a taskbar icon that says one thing and opens another. Duplicate entries for the same app are caught at creation. Editing the address of a pinned app already routes through the re-pin confirmation, and manifest apps inherit that unchanged.

### 6. Recognition rather than recall

One list, so provenance never has to be recalled in order to find something. The site's name is displayed rather than remembered. Search covers both names, so either one the user recalls will work.

### 7. Flexibility and efficiency of use

The in-scope start page and the manual-shortcut route give experienced users what they need without ever putting the word "manifest" in front of anyone else. The reset controls appear only when relevant.

### 8. Aesthetic and minimalist design

The whole feature is one badge, one sentence, one caption per constrained field, and two reset controls that hide themselves. No provenance card, no Type column, no filter bar, no separate sections, no advanced panel. Provenance gets the pixels its consequences justify and no more.

### 9. Help users recognize, diagnose and recover from errors

The out-of-scope message names what was wrong, what the rule is, and what to do instead, and the alternative is a link rather than an instruction. The rename notice is phrased as something that happened, with an action attached.

### 10. Help and documentation

The provenance sentence and the scope caption are the documentation, placed where the question arises. The existing "Learn more about Taskbar Tabs" link absorbs the longer explanation. No help modal, no tooltip carrying load-bearing information.

---

## 7. Deliberately not built

- **Two list sections, a Type column, or a provenance filter.** Costs recall and space to organise by a fact that matters on two screens.
- **A hidden advanced address override.** Adds only the spoofing case, and hides it where the wrong people will find it.
- **Editable icons, for either kind.** Already excluded; for a manifest app the icon is part of the identity claim, so the argument is stronger, not weaker.
- **A verified or trusted badge.** Firefox verifies nothing beyond origin control.
- **"Uninstall" as separate vocabulary.** Nothing extra is uninstalled today.
- **Blocking the rename of a manifest app.** §4.

---

## 8. Changes to the specification

| Section | Change |
|---|---|
| §2.1 Search | Also matches the site-provided name |
| §3.1 Row badges | Add **From the site** / **Added by you** |
| §4 Per-app settings | Add the provenance line to the header for manifest apps |
| §4.1 Name | Site-provided name shown beneath; **Use the name from the site** reset |
| §4.2 Address | Scope validation, error copy, and the add-a-Taskbar-Tab recovery link |
| §4.3 Use home page | Becomes **Use start page** for manifest apps |
| §5.1 Add dialog | App-or-shortcut branch when a manifest is found; duplicate detection |
| New §4.11 | This site provides an app: the upgrade row |
| New §4.12 | Renamed by the site: the information bar and **Keep calling it {old}** |
| New §9 | Where each value comes from: the provenance table from §1 |

Rationale gains one section: why the name is overridable and the address is not.

---

## 9. Open questions

1. **Badge wording.** "From the site" / "Added by you" is recommended on honesty grounds, over "Installed app" / "Shortcut" on recognition grounds. Worth testing; the two arguments are close and the familiar pair may simply win with real users.
2. **When Firefox starts reading manifests, and which members land first.** The distinction is downstream of that, and the point of settling it now is that the first app to arrive with a developer-owned name should meet an interface that already knows what to do with it.
3. **Scope trust for link capture.** A declared `scope` is a better capture rule than the host heuristic in §4.7, and it interacts with the open question already recorded about separating start page from capture scope. Manifest apps may resolve that question for free.
4. **Whether a manifest rename should rebuild the shortcut at all.** Rebuilding may cost the pin, so a developer's edit could silently unpin an app. Adopting the name only in-app, and rebuilding the shortcut on next explicit save, may be the safer trade. Needs the same real-hardware testing the existing pinning question needs.
5. **Enterprise.** Chromium lets administrators force-install apps. Force-installed entries would be a third provenance, and neither the badge pair nor the editing rules currently cover them.
