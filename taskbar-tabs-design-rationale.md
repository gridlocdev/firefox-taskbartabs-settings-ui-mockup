# Managing Taskbar Tabs: Design Rationale

A walkthrough of every decision in the "Manage Taskbar Tabs" mockup, and why it was made that way.

---

## The problem we're solving

Firefox can turn a website into a Taskbar Tab: its own window, its own icon, pinned to the taskbar. That part shipped in Firefox 143.

What's missing is everything *after* that. Once you've added a few, there's no list of them, no way to rename one, and no way to see which container it's using. Changing the page one opens to means hand-editing a JSON file in your profile folder. Getting rid of one means hunting for its icon.

This design adds the missing half: one place to see every Taskbar Tab you've made and change how each one behaves.

---

## Where it lives

**It's a sub-page of Settings, under Browsing.**

Acorn's settings guidance says new settings should fit into an existing category rather than getting a new one, and Browsing is where things like Tab management, Applications and Downloads already live. Taskbar Tabs belong with them.

It's a *sub-page* rather than a card on the main Browsing page because managing a list of installed things needs more room than a settings card gives you. Acorn recommends sub-pages for exactly this, and says they should carry a breadcrumb so people remember where they came from. Ours reads **Browsing › Taskbar Tabs**, and the app-level page adds a third step: **Browsing › Taskbar Tabs › Outlook**.

## Why "Taskbar Tabs" and not "Web Apps"

Both names are floating around. "Web apps" is what the support docs say; "Taskbar Tabs" is the name in Firefox's own source code and in Mozilla's public discussion of the feature.

We went with **Taskbar Tabs** throughout, in title case, treated as a proper feature name. Every reference follows it ("Add Taskbar Tab…", "No Taskbar Tabs yet", "Remove Taskbar Tab?") because a feature that calls itself two different things in the same screen teaches people nothing.

One exception: the address bar button is still called **Add to taskbar**, because that's its actual label in Firefox today, and the setting that hides it refers to it by name.

---

## The main page

### Three settings, not more

The Settings card has exactly three checkboxes:

- Show the "Add to taskbar" button in the address bar
- Open links in their Taskbar Tab when one exists
- Reopen Taskbar Tab windows when Firefox restarts

Everything else that could have been a global setting turned out to belong to individual tabs instead. There's no global "which page should a new tab open to?" preference, for example, because each tab now has its own editable address, and a global default would just be a second place to change the same thing.

**Why checkboxes rather than toggles.** Acorn has a rule for this: toggles turn a *feature* on and off, checkboxes pick a *configuration*. The giveaway is the label. "Taskbar Tabs" would be a toggle; "Reopen Taskbar Tab windows when Firefox restarts" is plainly a configuration choice. All three are configurations, so all three are checkboxes.

**Why there's no master on/off switch.** There was one early on, sitting above the others and dimming the whole page when off. It went because it muddied things: a switch that greys out a list of apps you can still see, plus shortcuts on your taskbar that quietly change behaviour, is a lot of consequence hiding behind one control. The three checkboxes cover what people actually want to change.

### Restore defaults sits on the heading row, and hides itself

This one we copied straight from Firefox.

In `about:preferences#home`, the Restore Defaults button sits at the end of the section heading row, and it's **invisible until something on the page differs from its default**. Rather than an accident, that was a specific UX request during development, and the reasoning in code review was blunt: otherwise you have a button that appears to do nothing when you click it.

So ours does the same. Change any of the three checkboxes and it appears; click it and it vanishes again. The space stays reserved while it's hidden, so nothing jumps around, and a hidden button can't be reached by keyboard either.

One difference from Firefox: theirs resets preferences beyond the section it appears in, which reviewers on that bug flagged as confusing. Ours only resets the three checkboxes directly beneath it, so what it does is exactly what you can see.

### The list itself

Each Taskbar Tab is a row showing its icon, name, address, when you last opened it, and a few small badges. Rows use Acorn's Box Group and Box Item components, the same pattern Firefox uses for saved addresses.

**Badges say what's true, not what's clickable.** "Pinned to taskbar", the container name and colour, "Opens at login", "Exact page". They're read-only labels. Everything you can *do* lives in the two controls at the end of the row.

**Search and sort appear because the list can get long.** Sort covers name, recently used, recently added and data stored. Neither Chrome nor Edge offers either, which is one of the few places this design is straightforwardly ahead of them.

**Select-all and bulk actions, but only one bulk action.** You can select several tabs and remove them together. You *can't* bulk-unpin them, for reasons covered further down.

### The row menu

Open, Open in new tab, Pin to taskbar, Rename…, Copy address, App settings…, Clear cookies and site data…, Remove Taskbar Tab…

Short labels because the surrounding context already says what these are. "Open" beats "Open web app" when the menu belongs to a row that's clearly a Taskbar Tab.

**Rename doesn't open a dialog.** It jumps to the app's settings page with the name field focused and selected. Renaming and changing the address are the same operation underneath (both rebuild the shortcut), so giving them one screen means one confirmation, one save, one mental model. A separate rename dialog would have been a second way to do half the job.

### Two empty states, two different messages

- **No Taskbar Tabs at all:** explains how to make one ("open a site you use often, then choose Add to taskbar in the address bar") and offers the Add button. An empty screen should tell you what to do next.
- **Search found nothing:** says what you searched for and offers to clear it. Different problem, different answer.

---

## The app settings page

### Name and address are a form, with Save and Cancel

Everywhere else on this page, changes apply the moment you make them. Name and address are the exception, and deliberately so.

Both of them rebuild the shortcut file that Windows uses to launch the app. Rather than a preference flip, that's an action with a consequence, so they get an explicit form: type freely, press **Save changes**, or press **Cancel** to throw the edits away. Both buttons stay disabled until you've actually changed something.

Validation happens on save, not while you type, so you're not scolded halfway through typing a URL.

### The save confirmation, and why it's blue rather than yellow

If the tab is pinned to your taskbar, saving pops a confirmation:

> **Save changes?**
> Outlook will be renamed to Work Mail.
> Work Mail will open the following URL:
> **https://outlook.office.com/calendar**

Three deliberate choices here.

**It uses the information style, not a warning.** Nothing is being lost or broken, and you may just have to pin the app again, which is worth telling someone without alarming them.

**The URL gets its own line, in bold.** Bold is Acorn's documented way to emphasise a line of body text. An earlier version put the URL in a bordered box, where it looked exactly like a text input sitting in a dialog full of text inputs, which is the wrong signal entirely. Plain bold text can't be mistaken for something you're supposed to fill in.

**It shows the URL Firefox will actually use.** Type `outlook.office.com/calendar` and the dialog shows `https://outlook.office.com/calendar`. What you see is what gets written to the shortcut.

If the tab isn't pinned there's nothing to warn about, so it saves immediately with no dialog.

### "Use home page" and its tooltip

Next to the address field is a button that trims the address back to just the site. It's disabled when you're already there.

Its explanation lives in a hover tooltip rather than a line of text under the field, so the form stays visually quiet. The tooltip names the actual site, as in "Shortens the address to just outlook.office.com", because a generic explanation is harder to act on than a specific one. It appears on keyboard focus as well as hover, and screen readers get it too.

### Container: editable, but with a confirmation

Containers were read-only at one point, on the grounds that switching one is disruptive. That was over-cautious, because people have legitimate reasons to move a tab between containers, and taking the option away doesn't remove the disruption; it just makes them delete and re-add the tab.

So it's a normal dropdown now. Picking a different container opens a confirmation that names both sides:

> Outlook will use the personal container's cookies and sign-ins for outlook.office.com, so you may be signed out of the app.

**"May be" rather than "will be"**, because whether you're actually signed out depends on what's already in that container.

**Cancelling actually reverts.** Escape, the Cancel button and clicking outside all snap the dropdown back to where it was. A dropdown showing a container the app isn't in would be worse than not offering the option at all.

**Confirming has visible consequences.** The stored-data figure drops to zero, matching what the dialog told you, and a toast offers Undo which restores both the container and the figure.

### Two behaviour toggles

**Open when I sign in** and **Open links to this site here** are genuine feature on/off switches for one app, so by Acorn's rule they're toggles, not checkboxes.

The link toggle depends on the global setting above it. When the global one is off, this one greys out and its description says where to turn it back on, rather than silently doing nothing.

### Permissions are shown, not edited

This is the most important honesty decision on the page.

Firefox grants permissions to a **site**, not to a Taskbar Tab. If you blocked the camera for `outlook.office.com`, it's blocked in the app *and* in every ordinary tab. There's no per-app version of that setting, and pretending otherwise would be a lie built into the interface.

So the page lists the four relevant permissions with their current state (Allowed, Ask every time, Blocked) under a caption that says plainly what the scope is. Changing them means following a link out to site permissions, where the change is honestly presented as site-wide.

We could have put editable dropdowns here, and they would have worked, but they would also have taught people something false about how Firefox works.

### Data and removal

The stored-data figure names both the site *and* the container, and says that tabs in the same container share it, because "how much does this app use?" is genuinely ambiguous when the same site is also open in a tab.

Removing a tab explains what actually happens: Firefox deletes the shortcut it created; the site stays available as a normal tab; your sign-in survives unless you tick the box to clear its data.

---

## The honesty problem: pinning

This shaped more of the design than anything else, so it's worth its own section.

**Windows decides what stays on the taskbar, not the browser.** Programmatic taskbar pinning was locked down years ago, so a browser can *ask* to add an icon but cannot take one away. Even Microsoft's own guidance for Edge tells users to right-click the taskbar icon and choose Unpin, because Edge cannot do it for them.

Early drafts of this design had a "Pinned to taskbar" toggle, and a bulk "Unpin from taskbar" button. Both were fiction. Flipping a switch that the operating system will ignore is worse than not offering the switch.

So:

- **There's no pin toggle.** "Pin to taskbar" appears in the row menu only when a tab isn't pinned, because adding is something Firefox can genuinely do.
- **There's no unpin action anywhere.** Not in the menu, not on the settings page, not in bulk.
- **The badge is informational.** "Pinned to taskbar" tells you the state; it isn't a control.
- **Removing a pinned tab warns you.** The remove confirmation adds a note: if the icon is still on your taskbar afterwards, right-click it and choose Unpin.
- **Renaming or re-addressing a pinned tab warns you**, for the same underlying reason: the shortcut is replaced, so the old pin may not survive.

The interface is quieter for it, and nothing in it promises something that won't happen.

---

## What we deliberately didn't build

- **A per-app on/off switch.** Removing a tab is the honest version of turning one off.
- **Change icon.** Icons come from the site. Overriding them is a bigger feature with its own storage and fallback questions.
- **A global "open at home page vs. exact page" preference.** Made redundant the moment each tab got an editable address.
- **Editable per-app permissions.** Covered above.
- **Anything mobile-specific.** Taskbar Tabs are a desktop feature. Android and iOS handle installed sites through entirely different OS mechanisms, and none of this design applies there.

---

## How it looks

Everything comes from Acorn's published tokens rather than invented values.

- **Colour:** accent `#0062fa` in light, `#00cadb` in dark; the four feedback backgrounds for message bars; light and dark themes throughout.
- **Type:** the in-content scale with a 15px base: 22px page headings, 17px card headings, 13px for descriptions and controls. System fonts, because Acorn specifies the OS font rather than a bundled one.
- **Space:** the in-content scale, 2 / 4 / 8 / 12 / 16 / 24 / 32.
- **Components:** Card, Box Group, Box Item, Box Link, Button, Checkbox, Toggle, Input Text, Input Search, Message bar, Badge, Breadcrumb.

Rather than invent a component when something wasn't in Acorn, we reached for a token. The emphasised URL in the save dialog is a good example: it started as a custom monospace code block, which was a made-up component, and ended up as Acorn's documented bold body text.

---

## Accessibility and keyboard use

- Visible focus rings using Acorn's focus colour, at 2px with 2px offset.
- Escape closes menus and dialogs; cancelling a container change reverts the dropdown.
- Hidden controls use `visibility: hidden`, which keeps them out of the tab order and off screen readers, so there are no invisible focus traps.
- The tooltip is a real tooltip with `role="tooltip"` and `aria-describedby`, and shows on keyboard focus, not just hover.
- Every icon-only button has a label naming the app it belongs to.
- Reduced motion is respected.
- Colour is never the only signal: container badges pair a colour dot with the container's name.

---

## How this compares to Chrome and Edge

Worth knowing where this lands.

**Things Chromium has that this matches:** open in window vs. tab, run on OS login, per-app link capture, permissions access, clear-data-on-uninstall.

**Things this has that Chromium doesn't:** renaming after install, editing the address after install, containers, search, sort, multi-select and bulk removal. Renaming in particular has been an open Edge feature request for years, since people installing the same site across several profiles end up with several identically-named apps.

**Things Chromium has that this deliberately skips:** a file-handling toggle, and enterprise policy surfaces.

---

## Open questions

1. **Scope vs. start page.** Right now the address does double duty: it's where the app opens *and*, by host, which links get captured. Firefox may eventually want these separate: a start page of `/mail/inbox` with a capture scope of the whole domain. That's one more field when it's needed.
2. **What happens to the pin when the shortcut is rebuilt.** We warn about it. Whether Windows actually drops the pin in practice needs testing on real hardware.
3. **macOS.** None of this addresses the Dock. macOS needs real `.app` bundles per app, which is a different shape of problem and the main reason the feature is still Windows-first.
