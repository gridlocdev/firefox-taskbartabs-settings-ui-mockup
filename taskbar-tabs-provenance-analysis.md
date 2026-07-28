# Tabs that a person adds, and web apps that a site declares: a design analysis

The current design treats each entry in the list as the same type of object: a name and an address that the user selected. That is true for a Taskbar Tab that a person added from the address bar. It is not true for a site that supplies a `.webmanifest`. For that site, the developer declares the name, the icon, the start page, and the area of the app. These values are the identity of the app, and not a preference of the user.

This document decides how the interface must show that difference. It then examines the result against the ten usability heuristics of Jakob Nielsen.

**Premise:** Taskbar Tabs in Firefox today takes a name and an icon from the page, and not from a manifest. Thus the second type of entry does not exist yet. This work is intentionally in front of that change. The interface for an identity that a developer owns must be correct before the first app comes with such an identity, and not after. §5 gives the experience of Chrome, which shows what occurs when a browser adds this function later.

**Status:** built. The prototype in [mockup.html](mockup.html) implements each part of it. The [specification](taskbar-tabs-options-specification.md) holds the behaviour: §3.1 for the badge, §4.1 and §4.2 for the edit rules, §4.11 and §4.12 for the two information bars, §5.1 for the add flow, and §9 for the summary.

---

## 1. The true differences

The source of an entry is not one difference. It is a set of differences, and only some of them cause a change in the interface.

| | Added by the user | Declared in a manifest |
|---|---|---|
| **Name** | The user typed it, or Firefox took it from the host | `name` or `short_name`. The developer owns it, and **it can change when Firefox gets the manifest again**. |
| **Address that it opens** | The URL that the user gave | `start_url` |
| **Area that it covers** | Calculated: the host of the address | `scope`, declared explicitly. The default is the directory of `start_url`. |
| **Identity after a change** | The row | `id`, which shows the difference between "the same app with a new name" and "a different app" |
| **Icon** | A favicon, or a letter tile | `icons`, with sizes and purpose tags |
| **Window shape** | The decision of Firefox | `display` (`standalone`, `minimal-ui`, and others) |
| **The meaning of a removal** | Delete a shortcut | Delete a shortcut. Later, also handler registrations, badging, and notification state. |

Two of these differences are sufficiently important to change the interface. The others are implementation details.

**The name can change without an action by the user.** No other value on this page has that behaviour. An app that the user pinned as "Outlook" can become "Microsoft Outlook" because a developer edited a JSON file. The interface must have an answer for that condition.

**The address is a claim about identity, and not a preference.** A window presents itself as an app, has the icon of the site, is on the taskbar, and holds the cookies and permissions of the site. Thus it makes a statement about what it is. If a person can point the address to a different location, that statement becomes a tool for deception: an entry with the name of Outlook and the icon of Outlook that opens a different site. The reason for the constraint is not that the developer declared the area. The reason is that the identity must stay true.

Each other difference, which is the icon, the window shape, and the removal mechanics, is invisible to the person who uses the page. It must stay invisible.

---

## 2. The recommendation in one table

| Field | Added by the user | From a manifest |
|---|---|---|
| **Name** | Free text | **Free text**, with the name from the site permanently below it and a **Use the name from the site** reset control |
| **Address** | Any valid `http(s)` URL | **Editable, and constrained to the area of the app.** The default is `start_url`. Firefox refuses an address outside the area, gives the reason, and gives an alternative. |
| **Icon** | Not editable | Not editable |
| **Each other value** (container, login, link capture, permissions, data, removal) | Unchanged | Unchanged |

This is different from the instinct to lock the name. §4 gives the argument.

---

## 3. How the list shows the difference

### One list, and not two

The obvious decision is to divide the list into "Installed apps" and "Shortcuts". This design refuses that decision.

A person who looks for Teams does not know how Teams came into the list, and must not have to know. Two sections make that person recall the source to find the item. This is the exact failure that heuristic 6 prevents. Two sections also make two empty states, break "Select all", and give the sort order two meanings at the same time. The difference is real, but it is a property of a row. It is not a method to group the page.

This design also examined filters and a Type column, and refused them. Six apps do not need a filter. A column uses permanent horizontal space for a fact that is important on about two screens.

### A badge, in the same style as the existing badges

The row already has a set of read-only labels: pinned, container, opens at login, and exact page. The source of an entry is the same type of fact. Thus it joins these labels.

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

Both states have a badge, and not only one state. This follows the example of **Pinned to taskbar** and **Not pinned**. A badge that is not there is a weak signal. A reader cannot tell the difference between "this app has no badge" and "I did not see the badge".

Manifest icons have full colour and a design. A manual entry falls back to a letter tile. Thus, in practice, the two types look different before a person reads a word. That is a useful pre-attentive signal, and it has no cost. But it cannot be the only signal, because this design never uses colour as the only signal.

### The text of the badges

**The recommendation is "From the site" and "Added by you".**

The second candidate was "Installed app" and "Shortcut", from Chromium and from mobile devices. That pair has true recognition value, and it lost. Firefox installs both types in the same manner, and writes the same shortcut file in the same manner. Thus the word "installed" for one of the two types states a mechanical difference that does not exist. This is the same type of untrue statement that this design already refused when it removed the unpin toggle.

"From the site" and "Added by you" give the source exactly. They also give the edit rules directly: the site defined the app, thus the site owns the name.

**The badge must not show approval.** It has no checkmark, no shield, and no "Verified" text. The party that controls the origin asserts the manifest, and Firefox does no more verification. A badge in the shape of a trust signal would be a security statement that the browser cannot support.

---

## 4. How the page for each app shows the difference

### The pattern is already on this page

The interface shows the site permissions (§4.8) but does not edit them. It gives one sentence that tells the reader that the true value is at the level of the site. It also gives a link to the location where a person can change the value honestly. The fields that a manifest owns are the same shape of problem, because the true value is with the site. Thus they must use the same pattern, and not a new pattern.

This is the strongest structural argument in the analysis. The source of an entry does not need a new visual language. It needs the language that the page already uses to say "this is the true state, and this is where the state lives".

### One sentence at the top

The interface shows this line below the app header, and only for a manifest app:

> Outlook provides its own name, icon and start page. You can rename it here for yourself.

It is not a card, not a banner, and not a dialog. It is one line of muted body text. It is in the position where a reader sees it before the first field that the reader cannot freely change.

### Name

```
Name
┌───────────────────────────────────────────────────────────┐
│ Outlook (Work)                                            │
└───────────────────────────────────────────────────────────┘
Shown in the Start menu and Alt+Tab. The site calls this app
"Microsoft Outlook".  [Use the name from the site]
```

The field stays editable. The name of the site is permanently below the field, and not only while an override is active. Thus the claim of the developer is always legible. The reset link shows only when the two names are different.

**Why the design does not lock the name.** The name is a private label. Firefox never transmits it, never uses it in a decision about an origin, and never uses it as a security boundary. Also, the rename function for an installed app is the function that users most frequently ask for in the equivalent Chromium feature. If you install one site in more than one profile, you get more than one icon with the same name, and this is the manifest condition exactly. To lock the name would make a dead end in the only scenario that truly needs the function, and would give no security advantage, because the risk of deception is fully in the address. To keep the claim of the developer, the interface shows the name from the site. It does not remove the name of the user.

### Address

```
Address
┌───────────────────────────────────────────┐  ┌──────────────────┐
│ https://outlook.office.com/mail/inbox     │  │ Use start page   │
└───────────────────────────────────────────┘  └──────────────────┘
```

The field accepts text and validates it. It is not disabled. Pages in the area are permitted. Thus "open Outlook at my calendar and not at my inbox" operates. Firefox refuses pages outside the area:

> `example.com` is outside this app's area. Outlook covers `outlook.office.com/mail/`. To open a different site in its own window, **add a Taskbar Tab** instead.

That message gives the problem, the rule, and an alternative that operates, in one message. The link puts the text that you typed into the Add dialog.

**Why the field has a constraint and not a lock.** A disabled field answers only "you cannot". The user must then guess if the field is defective, unsupported, or forbidden. A field with validation gives the rule at the moment that the rule is important, which is what heuristic 5 asks for. The rule is also exactly what the manifest specifies, because `scope` is the declared area of the app. Thus Firefox did not invent the constraint.

**The rule shows one time, and not permanently.** An earlier version put a line below the field: "Must stay within outlook.office.com/mail/, the area this app covers." That line gave the same information as the validation message, and it stayed on the page although almost each address is correct. The validation message keeps the information, gives it at the moment of the error, and adds the alternative. Thus the permanent line went away.

For **Use home page** (§4.3): on a manifest app, the button becomes **Use start page** and sets the address to `start_url`. That is the answer of the developer to the same question that the button asks.

---

### No advanced override

This design must not build an escape route for addresses outside the area, in a hidden position or in a visible position.

Each correct need already has an answer. You can edit pages in the area, and you can add each other URL as its own Taskbar Tab with two clicks. An override would add only the incorrect condition: an entry that keeps the name and the icon of an app but points to a different location. Also, in practice, hidden advanced flows are the most dangerous for the users who are the most easy to deceive. "Open settings, expand Advanced, paste this" is a script for a support fraud.

The escape route is a different object with an honest label. It is not a weak version of this object.

---

## 5. Flows that change

### To add a tab

The Add dialog already validates the address. When the address gives a site with a manifest, the dialog offers the choice and does not decide silently:

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

If you select the app, Firefox takes the name from the manifest and locks the address to `start_url`. If you select the shortcut, the two fields stay free. The button in the address bar uses the same logic: **Install Outlook** when a manifest exists, and **Add to taskbar** when no manifest exists.

### A site starts to supply an app

A manual shortcut to a site that later supplies a manifest gets one row on its settings page:

> **This site provides an app.** Outlook offers its own name, icon and start page.
> **Switch to the app…**

The change keeps the container, the login setting, and the link capture setting, and takes the identity from the manifest. It makes the shortcut again. Thus the existing confirmation about the pin (§5.2) applies without a change.

### The developer changes the name of the app

This condition has no answer today. It is also the condition that is the most probable to occur in the field and not in a review.

| Condition | Behaviour |
|---|---|
| The user set a name | Keep it. Never replace a local name. The "site calls this app…" line shows the new value. |
| The user set no name | Take the new name and make the shortcut again. |

Firefox never takes the new name silently. The settings page shows an information bar until you dismiss it:

> outlook.office.com renamed this app from **Outlook** to **Microsoft Outlook** on July 12.
> [Keep calling it Outlook]  [Dismiss]

The date has its year only when it is not the current year: **July 12**, or **July 12, 2025**.

"Keep calling it Outlook" writes a local override. That is the same mechanism as a manual rename. Thus this design adds no new state. The manifest `id` makes this a new name, and not a new app.

**Chrome built the same mechanism.** This is important, because it is independent evidence and not a borrowed idea. Chrome gives the problem as follows: *"Because Web Apps lack a central authority like Google Play to review updates, these modifications must be presented clearly to users for confirmation."* [Chrome 144](https://developer.chrome.com/blog/improvements-to-web-app-updates) divides the manifest. Icons and names are *"security sensitive members"* and Chrome holds them separately. Each other member applies immediately. A change of identity waits behind a **Review app update** item in the menu of the app. The version before it forced *"a disruptive dialog requiring them to either uninstall the app or immediately accept icon and name changes"*. This design also refuses that modal.

There is one difference. Chrome holds the change and asks the person to accept it. This design takes the change and offers the refusal. The local override makes that possible: to refuse is not to stop an update, it is to write a label. Thus no entry keeps an old identity that Firefox must compare with the manifest. The other rules from Chrome apply without a change. Icon differences below 10% apply automatically, and that rule belongs with an icon pipeline that this design does not have. Changes to `start_url` and to `scope` apply as configuration, and that rule keeps the address constraint with the app. The constraint does not stay at the value from the day of the installation. The rationale gives the longer version.

### To remove a tab

The removal keeps the name **Remove Taskbar Tab** for both types, with the same confirmation. Firefox removes the Taskbar Tab. It does not uninstall the manifest, and the site stays available in a usual tab. The existing dialog already says this. The word "Uninstall" would cause an expectation of a more complete removal than the true removal.

Examine this decision again if manifest apps get a true installed state: handler registrations, a separate storage bucket, or registration at the level of the operating system. "Uninstall" is then the honest word, and the confirmation must then give each item that the removal deletes.

### Search

The search compares the text with the name from the site and with the local name. A person who changed the name of an app to "Work Mail" can search for "Outlook". A message that says that there are no results, for an app that is in the list, is a bad failure.

### Duplicates

A manual shortcut and a manifest app with the same start URL must not both be possible without a message. The Add dialog must give this information and offer to open the existing entry.

---

## 6. Analysis against the heuristics

### 1. Visibility of system status

A row gives its type. A constrained field gives its constraint before a violation, and not after. Firefox announces a name that a developer changed, and the user does not have to find it.

The failure that this design prevents is the silent failure: an app with a new name that changed during the night, or a field that does not accept text for a reason that the page never gives.

### 2. Match between the system and the real world

The interface has no `manifest`, no `scope`, no `start_url`, and no `id`. It uses "the area this app covers", "the start page", and "the name from the site". It gives the rule for the area as the URL prefix that a reader can compare with the text that the reader typed. It does not give the rule as a concept from a specification.

"Added by you" and "From the site" are statements about the source of an item. A person can answer that question about their own computer.

### 3. User control and freedom

The user can replace the name and can reset it. The user can change the start page in the area of the app. The user can reach a URL outside the area as its own Taskbar Tab. The user can refuse a new name from a developer.

No branch of this design ends with "you cannot, and there is no alternative".

### 4. Consistency and standards

Internally, the source of an entry uses the read-only pattern with an explanation from Site permissions, and the two-state badge pattern from Pinned and Not pinned. Externally, the division between an installation and a shortcut agrees with Chromium, and this design uses the manifest fields as the specification gives them.

The violation that this design prevents is the most severe violation that is possible here: two rows that look the same but behave differently after a click.

### 5. Error prevention

Firefox validates the address against the area of the app and does not disable the field. Thus the user learns the rule at the point of use. The class of error that this truly prevents is not a typing error. It is an icon on the taskbar that says one thing and opens a different thing. Firefox finds duplicate entries for one app at the creation. An edit to the address of a pinned app already goes through the confirmation about the pin, and manifest apps use that confirmation without a change.

### 6. Recognition and not recall

There is one list. Thus a person never has to recall the source of an item to find it. The interface shows the name of the site, and the user does not have to remember it. The search compares the text with both names. Thus each name that the user recalls operates.

### 7. Flexibility and efficiency of use

The editable start page in the area, and the manual shortcut route, give experienced users what they need. The interface never shows the word "manifest" to the other users. The reset controls show only when they are applicable.

### 8. Aesthetic and minimalist design

The full feature is one badge, one sentence, one validation message that shows only after an error, and two reset controls that hide themselves. There is no card for the source of an entry, no Type column, no filter bar, no separate sections, and no advanced panel. The source of an entry uses the pixels that its consequences justify, and no more.

### 9. Help users to recognise, diagnose and recover from errors

The message for an address outside the area gives the error, the rule, and the alternative. The alternative is a link and not an instruction. The rename notice tells the user about an event, and gives an action with it.

### 10. Help and documentation

The sentence about the source and the validation message about the area are the documentation. They are in the position where the question occurs. The existing "Learn more about Taskbar Tabs" link holds the longer explanation. There is no help modal, and no tooltip that carries necessary information.

---

## 7. Not built, and the reasons

- **Two sections in the list, a Type column, or a filter for the source.** These cost recall and space, and they group the list by a fact that is important on two screens.
- **A hidden advanced override for the address.** It adds only the condition for deception, and it hides that condition where the wrong persons will find it.
- **Editable icons, for each type.** This design already refused them. For a manifest app, the icon is part of the identity claim. Thus the argument is stronger.
- **A verified badge or a trusted badge.** Firefox verifies nothing more than the control of the origin.
- **"Uninstall" as separate vocabulary.** Firefox uninstalls nothing more today.
- **A block on the rename function of a manifest app.** See §4.

---

## 8. Changes to the specification

| Section | Change |
|---|---|
| §2.1 Search | It also compares the text with the name from the site |
| §3.1 Row badges | Add **From the site** and **Added by you** |
| §4 Settings for each app | Add the line about the source to the header, for manifest apps |
| §4.1 Name | Show the name from the site below the field. Add the **Use the name from the site** reset control. |
| §4.2 Address | Add the validation against the area, the error text, and the link that adds a Taskbar Tab |
| §4.3 Use home page | It becomes **Use start page** for manifest apps |
| §5.1 Add dialog | Add the branch between an app and a shortcut when a manifest exists. Add the duplicate check. |
| New §4.11 | This site provides an app: the row that changes a shortcut to an app |
| New §4.12 | Renamed by the site: the information bar and **Keep calling it {old}** |
| New §9 | The source of each value: the table from §1 |

The rationale gets one more section: why the name is editable and the address is not.

---

## 9. Open questions

1. **The text of the badges.** The recommendation is "From the site" and "Added by you", because they are honest. "Installed app" and "Shortcut" are more familiar. It is better to test the two pairs, because the arguments are near to each other and the familiar pair can win with true users.
2. **When Firefox starts to read manifests, and which members come first.** This difference is downstream of that change. The reason to decide it now is that the first app with a name that a developer owns must find an interface that already knows what to do with it.
3. **Trust in the declared area for link capture.** A declared `scope` is a better capture rule than the host rule in §4.7. It also interacts with the existing open question about the difference between a start page and a capture area. Manifest apps can answer that question without more work.
4. **If a manifest rename must make the shortcut again.** A new build can cause the loss of the pin. Thus an edit by a developer can remove an app from the taskbar without a message. It can be safer to take the name in the app only, and to make the shortcut again at the next explicit save operation. This question needs the same test on true hardware as the existing question about the pin.
5. **Enterprise devices.** Chromium lets administrators force an installation. A forced entry is a third source. The pair of badges does not cover it, and the edit rules do not cover it.
