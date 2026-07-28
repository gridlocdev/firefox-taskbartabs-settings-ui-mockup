# Managing Taskbar Tabs: Options Specification

Every configurable option in the interface, what each of its states does, and how it behaves at the edges.

**Location:** Settings → Taskbar Tabs (`about:preferences#taskbarTabs`), its own sidebar item directly below Tabs and browsing
**Platform:** Windows desktop (Linux behaviour follows where supported; macOS not covered)

---

## 1. Global settings

Three checkboxes in the Settings card. All apply to Firefox as a whole, for the current profile.

### 1.1 Show the "Add to taskbar" button in the address bar

| | |
|---|---|
| **Control** | Checkbox |
| **Default** | On |

| State | Behaviour |
|---|---|
| **On** | The "Add to taskbar" icon appears at the right of the address bar on every eligible page. Clicking it adds the current tab as a Taskbar Tab. On a site that provides an app (§9) the button reads **Install {name}** instead. |
| **Off** | The icon is hidden. Taskbar Tabs can still be added from the tab context menu and from the **Add Taskbar Tab…** button on this page. |

**This is the only place an app can be installed from.** The **Add Taskbar Tab…** button makes shortcuts and never installs, because installing requires the manifest and this is the one entry point where the page is already loaded (§5.1). Turning the button off therefore removes the install path, leaving §4.11 as the way to reach it: add a shortcut, then switch.

![The address bar button checkbox, turned on](docs/screenshots/spec/1.1-show-urlbar-button.png)

Turning this off never removes existing Taskbar Tabs, and never blocks adding new ones, since it only hides one entry point.

---

### 1.2 Open links in their Taskbar Tab when one exists

| | |
|---|---|
| **Control** | Checkbox |
| **Default** | On |

| State | Behaviour |
|---|---|
| **On** | Link capture is available. A link to a site you've added opens in that tab's window instead of a new browser tab, for each app that has its own switch on. |
| **Off** | Link capture is disabled everywhere. All links open in ordinary browser tabs regardless of per-app settings. |

![The link capture checkbox, turned on](docs/screenshots/spec/1.2-open-links.png)

**Interaction with the per-app switch:** this is the master for §4.7. When off, every app's "Open links to this site here" toggle is disabled and shows the message *"Turn on 'Open links in their Taskbar Tab' in Taskbar Tabs settings to use this."* Per-app values are remembered, not erased, and take effect again when this is turned back on.

**New tabs** inherit this setting as the initial value of their per-app switch.

---

### 1.3 Reopen Taskbar Tab windows when Firefox restarts

| | |
|---|---|
| **Control** | Checkbox |
| **Default** | Off |

| State | Behaviour |
|---|---|
| **On** | Taskbar Tab windows are included in session restore and come back when Firefox restarts. |
| **Off** | Taskbar Tab windows are excluded from session restore, matching Firefox's current behaviour. Ordinary tabs and windows are unaffected either way. |

![The session restore checkbox, turned off](docs/screenshots/spec/1.3-reopen-on-restart.png)

This does not control launching at sign-in, which is per-app (§4.6).

---

### 1.4 Restore defaults

| | |
|---|---|
| **Control** | Button, on the Settings card heading row |
| **Visibility** | Hidden while all three checkboxes are at their defaults; visible as soon as any one differs |

| Action | Result |
|---|---|
| **Click** | Sets §1.1 to On, §1.2 to On, §1.3 to Off. Shows the toast *"Taskbar Tab settings restored to defaults"*. The button then hides itself. |

![The Settings card with Restore defaults on the heading row](docs/screenshots/spec/1.4-restore-defaults.png)

*Shown with §1.3 changed from its default, which is what makes the button appear.*

**Scope:** the three checkboxes above it only. It does not touch any individual Taskbar Tab, its container, its address, its permissions or its data.

While hidden the button reserves its space (so nothing shifts) and is excluded from the tab order and from screen readers.

---

## 2. List controls

### 2.1 Search

| | |
|---|---|
| **Control** | Search field |
| **Default** | Empty |

Filters the list live as you type, matching against the tab's name, its address, and the name the site provides (§9). Case-insensitive.

Matching the site-provided name matters because an app renamed to "Work Mail" is still the thing someone will search for as "Outlook".

| Condition | Result |
|---|---|
| **Empty** | All Taskbar Tabs shown. |
| **Matches found** | Only matching rows shown; sort order preserved. |
| **No matches** | Empty state: *"No Taskbar Tabs match "…""* with a **Clear search** button. |

![The list filtered by a search term](docs/screenshots/spec/2.1-search.png)

*Searching for `home`, with one row matching.*

Changing the search clears any active selection.

---

### 2.2 Sort by

| | |
|---|---|
| **Control** | Dropdown |
| **Default** | Name (A to Z) |

| Option | Order |
|---|---|
| **Name (A to Z)** | Alphabetical by display name. |
| **Recently used** | Most recently opened first. |
| **Recently added** | Most recently added first. |
| **Data stored** | Largest stored data first. |

![The search field and sort dropdown](docs/screenshots/spec/2.2-sort-by.png)

Sorting affects display only. It has no effect on taskbar order, which Windows controls.

---

### 2.3 Select all / selection

| | |
|---|---|
| **Control** | Checkbox above the list, plus one per row |
| **Default** | Nothing selected |

| State | Behaviour |
|---|---|
| **Nothing selected** | The "Select all" checkbox is shown. |
| **One or more selected** | "Select all" is replaced by the bulk action bar reading *"N selected"*, with **Remove** and **Cancel**. |

![The list with two rows selected](docs/screenshots/spec/2.3-selection.png)

*Two rows selected, with the bulk action bar in place of the "Select all" checkbox.*

**Select all** applies to currently *visible* rows, so if a search is active it selects the filtered set only. Clicking it when everything visible is already selected clears the selection.

Selection is cleared by: Cancel, changing the search, or completing a removal.

---

### 2.4 Bulk Remove

| | |
|---|---|
| **Control** | Button in the bulk action bar |

Opens the remove confirmation (§5.4) for every selected tab at once. There is no bulk unpin (see §6).

![The bulk action bar](docs/screenshots/spec/2.4-bulk-remove.png)

---

## 3. Row actions

Each row has an **Open** button and an overflow (⋯) menu.

| Item | Availability | Behaviour |
|---|---|---|
| **Open** | Always | Launches the tab in its own window at its configured address. |
| **Open in new tab** | Always | Opens the same address as an ordinary browser tab, in the tab's container. |
| **Pin to taskbar** | Only when the tab is **not** pinned | Asks Windows to add the icon to the taskbar. |
| **Rename…** | Always | Opens the app settings page with the Name field focused and its text selected. |
| **Copy address** | Always | Copies the tab's full address to the clipboard. |
| **App settings…** | Always | Opens the app settings page. |
| **Clear cookies and site data…** | Always | Opens the clear-data confirmation (§5.5). |
| **Remove Taskbar Tab…** | Always | Opens the remove confirmation (§5.4). |

![A row with its overflow menu open](docs/screenshots/spec/3-row-menu.png)

There is deliberately **no unpin option**. See §6.

### 3.1 Row badges (read-only)

| Badge | Meaning |
|---|---|
| **From the site** | The app's name, icon and start page come from the site (§9). |
| **Added by you** | The address and name were chosen by the person, not the site. |
| **Installed by {authority}** | Force-installed by policy. Identity comes from the manifest as above, but removal does not (§4.10). |
| **Pinned to taskbar** | The icon is currently on the taskbar. |
| **Not pinned** | The tab exists but has no taskbar icon. |
| *Container name* + colour dot | The container the tab runs in. Absent when no container is set. |
| **Opens at login** | "Open when I sign in" is on for this tab. |
| **Exact page** | The address includes a path, so it opens a specific page rather than the site's home page. |

These are labels, not controls.

![A single row showing its badges](docs/screenshots/spec/3.1-row-badges.png)

---

## 4. Per-app settings

Reached by clicking a tab's name, **App settings…**, or **Rename…**.

For a tab whose identity comes from the site (§9), the header carries one extra line:
*"This app is provided by {host}, which sets its name, icon and start page. You can rename it here for yourself."*
Nothing else on the page changes shape.

### 4.1 Name

| | |
|---|---|
| **Control** | Text field, part of the Name and address form |
| **Default** | The name given when the tab was added, or the site-provided name |
| **Validation** | Cannot be empty |

Sets the name shown in the Start menu, in Alt+Tab and on this page. Does not change the site.

Changes are held until **Save changes** (§4.4). Submitting an empty name focuses the field and shows *"Give the app a name so you can find it in the Start menu"*.

**Editable for both kinds.** The name is a private label: it is never transmitted, never used in an origin decision and never a security boundary, so a site-provided name can be replaced with a local one.

| Element | Shown when |
|---|---|
| *"The site calls this app "{site name}"."* appended to the field description | The tab came from the site |
| **Use the name from the site** button | The current name differs from the site's |

The site's name stays visible whether or not it is in use, so the developer's claim is always legible.

![The Name field](docs/screenshots/spec/4.1-name.png)

---

### 4.2 Address

| | |
|---|---|
| **Control** | Text field, part of the Name and address form |
| **Default** | The address the tab was added with |
| **Validation** | Must resolve to a valid `http://` or `https://` URL |

The address the tab opens to. A bare host (`example.com`) opens the site's home page; a full address (`example.com/mail/inbox`) opens that page, which is what makes two pages on one site into two separate Taskbar Tabs.

| Input | Result on save |
|---|---|
| **Full URL** | Used as typed, normalised. |
| **No scheme** | `https://` is added automatically. |
| **Not a URL** | Save is blocked; the field is marked invalid and a message explains the expected format. |

Whether the tab shows the **Exact page** badge is derived from this field rather than from a separate switch.

**Constrained for tabs that came from the site.** The field stays editable, but the address must fall inside the area the app declares, so a description sits beneath it: *"Must stay within `outlook.office.com/mail/`, the area this app covers."*

| Input | Result on save |
|---|---|
| **Inside the area** | Saved normally. Opening at `/mail/calendar` instead of `/mail/inbox` is allowed |
| **Outside the area** | Rejected. The field is marked invalid and shows *"`example.com/inbox` is outside this app's area. {name} covers `outlook.office.com/mail/`. To open a different page in its own window, add a Taskbar Tab."*, followed by an **Add a Taskbar Tab instead** button that opens §5.1 prefilled with what was typed |

The field is validated rather than disabled. A disabled field says only "you can't"; a validated one teaches the rule at the moment it matters and leaves a working alternative in reach.

There is **no advanced override**. Every legitimate need is met by editing within the area or by adding a separate Taskbar Tab; an override would add only the case where an app keeps its name and icon while pointing somewhere else.

![The Address field](docs/screenshots/spec/4.2-address.png)

*The Address field of an app from the site, with the area it covers stated beneath it.*

![The Address field rejecting an out-of-scope address](docs/screenshots/spec/4.2-out-of-scope.png)

*The rejection names what was typed, the rule, and what to do instead.*

---

### 4.3 Use home page / Use start page

| | |
|---|---|
| **Control** | Button beside the Address field |
| **Label** | **Use home page** for a tab you added; **Use start page** for one from the site |

| Kind | Enabled when | Result |
|---|---|---|
| **Added by you** | The address includes a path | Trims the address back to its origin |
| **From the site** | The address differs from the app's start page | Resets it to the start page the app declares |

It edits the draft only, so you still have to save.

A hover/focus tooltip explains it, naming the target: *"Shortens the address to just outlook.office.com. Keep a full address to give one page its own Taskbar Tab."*, or *"Resets the address to https://outlook.office.com/mail/, the start page this app declares."*

![The Use home page button with its tooltip visible](docs/screenshots/spec/4.3-use-home-page.png)

---

### 4.4 Save changes / Cancel

| | |
|---|---|
| **Controls** | Two buttons below the Name and address fields |
| **Enabled when** | Either field differs from the saved value |
| **Disabled when** | Nothing has changed |

| Action | Result |
|---|---|
| **Save changes** (tab is **not** pinned) | Applies immediately. Toast: *"Changes saved for {name}"*. |
| **Save changes** (tab **is** pinned) | Opens the save confirmation (§5.2) first. |
| **Cancel** | Discards both edits and restores the saved values. |

Leaving the page also discards unsaved edits.

![The Name and address card with Save changes enabled](docs/screenshots/spec/4.4-save-changes.png)

*Both buttons enable only once a field differs from the saved value.*

---

### 4.5 Container

| | |
|---|---|
| **Control** | Dropdown |
| **Default** | The container the tab was created in |
| **Options** | No container, Work, Personal, Banking, Shopping (mirrors the profile's containers) |

Determines which cookies and sign-ins the tab uses.

| Action | Result |
|---|---|
| **Select a different container** | Opens the change-container confirmation (§5.3) immediately. |
| **Confirm** | Container is applied, the stored-data figure resets to zero, and a toast offers **Undo**. |
| **Cancel / Escape / click outside** | The dropdown snaps back to its previous value; nothing changes. |

Applies immediately on confirm, so it is not part of the Save changes form.

![The Container dropdown](docs/screenshots/spec/4.5-container.png)

---

### 4.6 Open when I sign in

| | |
|---|---|
| **Control** | Toggle |
| **Default** | Off for new tabs |

| State | Behaviour |
|---|---|
| **On** | The tab starts automatically when you sign in to the device. |
| **Off** | It starts only when you open it. |

Independent of §1.3, which governs Firefox restarts rather than device sign-in.

![The Open when I sign in toggle](docs/screenshots/spec/4.6-open-at-sign-in.png)

---

### 4.7 Open links to this app here

| | |
|---|---|
| **Control** | Toggle |
| **Default** | Inherits §1.2 at the time the tab was added, except for a second copy in another container, which starts off |
| **Disabled when** | §1.2 is off |

| State | Behaviour |
|---|---|
| **On** | Links falling inside this app's area, opened in Firefox, come to this window instead of a new tab. |
| **Off** | Those links open as ordinary tabs. |
| **Disabled** | Shows *"Turn on 'Open links in their Taskbar Tab' in Taskbar Tabs settings to use this."* The stored value is kept. |

**What counts as the app's area** depends on where the tab came from (§9):

| Kind | Area matched | Why |
|---|---|---|
| **From the site** | The declared scope, e.g. `outlook.office.com/mail/` | The site said what it covers, so nothing wider is claimed |
| **Added by you** | The whole host | A shortcut declares nothing, so the host is the only rule available |

Host matching is the coarser of the two, and on a shared host it captures other people's pages. The description says so: *"A shortcut has no declared area, so this covers the whole site."*

**Only one copy can capture.** Two copies of one app in different containers would otherwise both claim the same link and a link has one destination. Turning this on releases it from any sibling copy, and the description names the copy that holds it: *"Microsoft Outlook in work opens these links today. Turning this on moves them here."* A toast confirms the move.

![The Open links to this site here toggle](docs/screenshots/spec/4.7-open-links-here.png)

*Shown in its enabled state. When §1.2 is off, the toggle greys out and the description changes.*

---

### 4.8 Site permissions (read-only)

Four rows (Notifications, Camera, Microphone, Location), each showing **Allowed**, **Ask every time** or **Blocked**.

**Not editable here, by design.** Firefox grants permissions to a site, not to a Taskbar Tab, so these values apply to the host everywhere in the profile, including ordinary tabs. The page says so explicitly.

**Change permissions for {host}** links out to site permissions, where the change is presented at its true, site-wide scope.

![The read-only Site permissions card](docs/screenshots/spec/4.8-site-permissions.png)

---

### 4.9 Clear data

| | |
|---|---|
| **Control** | Button in Data and removal |

Shows the amount stored for the host in this tab's container, and notes that tabs using the same container share it. Opens the clear-data confirmation (§5.5).

![The Cookies and site data row](docs/screenshots/spec/4.9-clear-data.png)

---

### 4.10 Remove Taskbar Tab

| | |
|---|---|
| **Control** | Button in Data and removal |
| **Absent when** | The tab was installed by policy (§9) |

Opens the remove confirmation (§5.4).

Wording is the same for both kinds. What is removed is the Taskbar Tab: the site is not uninstalled, and it stays available in a normal tab, which is what the dialog already says.

**For a managed tab the button is absent rather than disabled**, and the row says who decides instead: *"Your organisation installed Microsoft Teams and manages whether it can be removed."* A disabled button with no explanation is the dead end §4.2 argues against; naming the authority answers the question the button would have raised. The same tab is also left out of selection entirely (§2.3), since removal is the only bulk action.

Removal deletes the shortcut and nothing else today. If Firefox ever registers file or protocol handlers from a manifest (§9), this section and §5.4 have to enumerate what else goes, because the current wording would then be false.

![The Remove this Taskbar Tab row](docs/screenshots/spec/4.10-remove.png)

---

### 4.11 This site provides an app

| | |
|---|---|
| **Control** | Information bar at the top of the Name and address card |
| **Shown when** | The tab was added by hand, and its site turns out to provide an app |

Reads *"**This site provides an app.** {host} offers {name} with its own name, icon and start page. Switching keeps your container, startup and link settings."*

| Action | Result |
|---|---|
| **Switch to the app…** | Adopts the site's name, icon, start page and area. Rebuilds the shortcut, so a pinned tab gets the save confirmation (§5.2) first. A name the person chose is kept, becoming a local override |
| **Not now** | Hides the bar for this tab |

![The information bar offering to switch to the app a site provides](docs/screenshots/spec/4.11-site-app.png)

---

### 4.12 Renamed by the site

The one value on this page that can change without the person doing anything.

| Situation | Behaviour |
|---|---|
| A local name is set | Keep it. The site's new name appears in the field description and nowhere else |
| No local name | Adopt the new name and rebuild the shortcut |

Adoption is never silent. An information bar sits at the top of the Name and address card until dismissed: *"{host} renamed this app from **{old}** to **{new}** on {date}."*

The date reads **July 12** for a rename in the current year and **July 12, 2025** for one before it, so the year takes up space only when it is not the obvious one.

| Action | Result |
|---|---|
| **Keep calling it {old}** | Writes {old} as a local name, using the same mechanism as a manual rename, and dismisses the bar |
| **Dismiss** | Keeps the new name and hides the bar |

![The information bar announcing a rename by the site](docs/screenshots/spec/4.12-renamed.png)

---

## 5. Dialogs

### 5.1 Add a Taskbar Tab

Opened by **Add Taskbar Tab…**.

| Field | Default | Notes |
|---|---|---|
| **Address** | Empty, required | The tab opens at exactly this address. Invalid input blocks submission with an inline error. |
| **Name** | Empty | If left blank, the host is used with any leading `www.` stripped. |
| **Container** | No container | Set once, here. |
| **Pin to taskbar** | Checked | Asks Windows to add the icon. |

**This dialog makes shortcuts only.** It never produces an app whose identity came from a manifest, and it never fetches the address that was typed.

The reason is the entry point. Installing from here would mean the Settings page issuing a request to an arbitrary, possibly pasted address before the person had committed to anything, which is a request they did not ask for to a server they may not know. Installing therefore happens from the address bar (§1.1), where the page and its manifest are already loaded because they chose to visit. A shortcut that turns out to point at a site providing an app is offered the switch on its own settings page (§4.11), which needs no speculative fetch either.

Each entry point does one thing, and neither has to guess.

**Duplicates** are caught on submit, and depend on the container, because two containers running one site is what containers are for:

| Case | Result |
|---|---|
| **Same address, same container** | Refused. *"{name} already opens this address in the same container. Open it from the list, or choose a different page."* |
| **Same address, different container** | Allowed, and the dialog says so before submission, since two copies share a name and an icon and the taskbar shows neither the container nor this list |
| **Same address, different container, same name** | Refused on the Name field. *"{name} is already the name of this app in {container}. Give this one a different name."* |

For the allowed case an information bar appears: *"You already have Microsoft Outlook in work. Give this one its own name so you can tell them apart on the taskbar."* The Name field is prefilled with a suggestion, `Microsoft Outlook (Personal)`, or `Microsoft Outlook (2)` when the new copy has no container. The suggestion follows the container as it changes and is dropped if the address stops matching, but a name that was typed is never replaced.

A second copy also starts with link capture off (§4.7), since only one copy can hold it.

On success the tab is added to the top of the list and a toast confirms, offering **Open**.

![The Add a Taskbar Tab dialog](docs/screenshots/spec/5.1-add-dialog.png)

![The Add dialog asking for a distinct name for a second copy](docs/screenshots/spec/5.1-second-copy.png)

*A second copy of an app that already exists in another container.*

---

### 5.2 Save changes?

Shown only when saving a **pinned** tab's name or address.

| Element | Shown when |
|---|---|
| *"{old name} will be renamed to {new name}."* | The name changed |
| *"{new name} will open the following URL:"* + the URL in bold | The address changed |
| Information bar: *"You may need to pin the app again, since this replaces the shortcut."* | Always (in this dialog) |

| Action | Result |
|---|---|
| **Save changes** | Applies both edits. |
| **Cancel / Escape** | Returns to the form with edits intact. |

The URL shown is the normalised version that will be written to the shortcut.

![The Save changes confirmation dialog](docs/screenshots/spec/5.2-save-dialog.png)

*Shown with both the name and the address changed.*

---

### 5.3 Change container?

| Element | Content |
|---|---|
| **Title** | *"Move {name} to {container}?"*, or *"Stop using a container for {name}?"* when moving to no container |
| **Body** | *"{name} will use the {container} container's cookies and sign-ins for {host}, so you may be signed out of the app."* |
| **Information bar** | Shown only when the tab is pinned: *"You may need to pin the app again, since this replaces the shortcut."* |

| Action | Result |
|---|---|
| **Move to {container}** | Applies; stored data resets to zero; toast offers **Undo**. |
| **Cancel / Escape / outside click** | Reverts the dropdown. |

![The Change container confirmation dialog](docs/screenshots/spec/5.3-container-dialog.png)

---

### 5.4 Remove Taskbar Tab?

| Element | Content |
|---|---|
| **Title** | *"Remove {name}?"* or *"Remove {n} Taskbar Tabs?"* |
| **Body** | Explains that Firefox deletes the shortcut it created, the site stays available as a normal tab, and the sign-in survives unless data is cleared. |
| **Checkbox** | *"Also clear cookies and site data for this site"*, default off |
| **Information bar** | Shown when any affected tab is pinned: *"If the icon is still on your taskbar afterwards, right-click it and choose Unpin from taskbar."* |

| Action | Result |
|---|---|
| **Remove** | Removes the tab(s); toast offers **Undo**, which restores them in their original list positions. |
| **Keep it / Escape** | Nothing changes. |

Removing the tab you're currently viewing returns you to the list.

![The Remove Taskbar Tab confirmation dialog](docs/screenshots/spec/5.4-remove-dialog.png)

---

### 5.5 Clear cookies and site data?

Body names the amount, the host, and warns that tabs using the same container are affected and you'll be signed out.

| Action | Result |
|---|---|
| **Clear data** | Stored data resets to zero; toast confirms. **Not undoable.** |
| **Cancel / Escape** | Nothing changes. |

![The Clear cookies and site data confirmation dialog](docs/screenshots/spec/5.5-clear-data-dialog.png)

---

## 6. Taskbar pinning: what is and isn't possible

Windows controls the taskbar, not the browser. Firefox can request that an icon be added; it cannot remove one.

| Capability | Available | Where |
|---|---|---|
| Add an icon to the taskbar | Yes | Add dialog, row menu **Pin to taskbar** |
| See whether a tab is pinned | Yes | Row badge |
| Remove an icon from the taskbar | **No** | Right-click the icon on the taskbar → **Unpin from taskbar** |
| Bulk unpin | **No** | n/a |

Because of this there is no pin toggle, no unpin menu item and no bulk unpin. The interface warns in two places where a pin may be affected: when saving a pinned tab's name or address, and when removing a pinned tab.

![The row menu for an unpinned tab](docs/screenshots/spec/6-pinning.png)

*The menu for an unpinned tab offers **Pin to taskbar**, and no unpin action exists anywhere in the interface.*

---

## 7. Feedback and undo

Toasts appear at the bottom of the window, dismiss automatically after about eight seconds, and can be dismissed manually.

| Action | Undoable |
|---|---|
| Remove (single or bulk) | Yes, restores to original list positions |
| Change container | Yes, restores container and stored-data figure |
| Add a Taskbar Tab | No, but the toast offers **Open** |
| Save name / address | No, because the confirmation dialog is the checkpoint |
| Switch to the app a site provides | No, for the same reason |
| Decline a rename by the site | No, but it only writes a name you can change again |
| Clear cookies and site data | No |
| Restore defaults | No |
| Pin to taskbar | No |

![A toast offering Undo](docs/screenshots/spec/7-undo-toast.png)

---

## 8. Page states

| State | Trigger | What's shown |
|---|---|---|
| **Normal** | One or more Taskbar Tabs exist | Settings card and the full list |
| **Empty** | No Taskbar Tabs at all | Settings card, plus an empty state explaining how to add one and an **Add Taskbar Tab…** button |
| **No search results** | Search matches nothing | The message and a **Clear search** button; the list is otherwise intact |
| **Selection active** | One or more rows selected | Bulk action bar replaces the "Select all" checkbox |
| **App settings** | A tab is opened | Sub-page with an **All Taskbar Tabs** back button. There are no breadcrumbs, matching the rest of Settings |

![The empty state](docs/screenshots/spec/8-empty-state.png)

*Shown when no Taskbar Tabs exist.*

---

## 9. Where each value comes from

Three kinds of entry share the list. One was typed in by hand; one was declared by the site in a manifest, where the name, icon, start page and area belong to the developer; one was installed by policy and takes its identity from a manifest the same way, but is not the person's to remove.

| | Added by you | From the site | Installed by policy |
|---|---|---|---|
| **Name** | Yours | The site's, replaceable with a local one (§4.1) | Same as From the site |
| **Address it opens** | Any valid `http(s)` URL | The declared start page, editable within the app's area (§4.2) | Same |
| **Area it covers** | Inferred: the host of the address | Declared by the site | Same |
| **Icon** | Favicon | From the site | Same |
| **Can change on its own** | No | Yes: the name, when the site is updated (§4.12) | Yes |
| **Can be removed** | Yes | Yes | No (§4.10) |

### An app's identity is the manifest plus the container

Chromium keys an installed app on its manifest `id` within a profile. Firefox's isolation unit sits inside the profile, so the same key would make two containers running one app indistinguishable to everything downstream of it.

The identity key is therefore the manifest `id` **and** the container. Consequences, all of them already specified:

| Question | Answer |
|---|---|
| Can one app exist twice? | Yes, once per container (§5.1) |
| How are they told apart? | By name, which the second copy is required to make distinct, because the taskbar shows neither the container nor this list |
| Which one captures a link? | Exactly one, chosen explicitly (§4.7) |
| Which one starts at sign-in? | Either, both, or neither. Two windows at sign-in is a coherent thing to want (§4.6) |

### Manifest identity requires a secure context

An app's name, icon, start page and area are only taken from a manifest served over HTTPS. A manifest fetched over plain HTTP is editable by anyone on the network, and an app's identity is precisely what should not be.

A **shortcut** to an `http://` address stays allowed, because addressing a device on your own network is a real thing to want and `http://homeassistant.local:8123` is a real example of it. Settings does not badge or warn about this. The warning belongs in the window, where the risk is (§10), not in a list of things you configured on purpose.

### The area is a set, not an origin

An app's area is stored as a list of matchers, currently always of length one. Chromium ships `scope_extensions`, which lets an app claim further origins if each one opts in by hosting a file naming the app. Whether or not Firefox implements it, the area must not be written as a single-origin comparison, or a legitimate multi-origin app would have its own second origin rejected by §4.2.

![A row showing the From the site badge alongside the others](docs/screenshots/spec/3.1-row-badges.png)

They share one list rather than two sections. Someone looking for Teams should not have to know how Teams arrived in order to find it. The difference is a property of a row, shown as a badge (§3.1), not an organising principle for the page.

**The badge is not a trust signal.** A manifest is asserted by whoever controls the origin, and Firefox verifies nothing beyond that, so **From the site** carries no checkmark, shield or "Verified".

### Why the name is editable and the address is not

The name is a private label with no security role, and renaming an installed app is the thing most often missing from equivalent features elsewhere, since installing one site across several profiles leaves several identically named icons. Locking it would buy nothing.

The address is different. A window that presents itself as an app, wears the site's icon, sits on the taskbar and holds the site's cookies and permissions is making a claim about what it is. Letting it be repointed anywhere is how that claim becomes a lie. So the address is held to the area the app declares, which is the site's own statement about its extent, not a restriction Firefox invented.

### Not built

- Two list sections, a Type column, or a provenance filter.
- A hidden advanced override for the address (§4.2).
- Editable icons, for any kind.
- A verified or trusted badge.
- Separate "Uninstall" vocabulary (§4.10).
- An insecure-origin badge in the list. The window carries it instead (§10).
- **File and protocol handlers.** Chromium registers the `file_handlers` and `protocol_handlers` a manifest declares into the Windows registry, and unregisters them on uninstall. This design models neither, because a claim on `.pdf` or on a URL scheme is an OS-level assertion Firefox would have to actually make, and a control for something the operating system will ignore is the unpin toggle again (§6). If they ever land, §4.10 and §5.4 must enumerate what removal takes with it, because "deletes the shortcut Firefox created" would no longer be the whole truth.
- **Anything a manifest declares beyond identity**: `share_target`, `shortcuts`, `launch_handler`, badging, widgets. None affects what this page shows.

---

## 10. The app window

The settings on this page produce a window with reduced chrome. That window is not part of Settings, but it is where the identity rules in §9 are either upheld or wasted, so its requirements belong here.

**The problem.** Removing the address bar removes the browser's main anti-phishing affordance. Constraining the address in §4.2 stops a Taskbar Tab being repointed through Settings; it does nothing about the app navigating itself somewhere else once it opens, which is cheaper and does not involve Settings at all.

**The rule.** The origin is always present, in real window chrome that content cannot paint over, and its appearance changes when the app leaves the area it declares.

| State | Shows |
|---|---|
| **In the app's area** | A padlock and the origin, in the title bar beside the app name |
| **Outside it** | A warning icon, the new origin, and *"Outside {app name}"*, replacing the calm state |
| **Not secure** | A warning icon and *"Not secure"* before the origin. This is where an `http://` shortcut is called out (§9), and it is the only place |

Chromium shows nothing while in scope and raises a mini URL bar only on leaving it. That is one state better than nothing and one worse than this, because a page rendered in standalone mode can draw a convincing imitation of the browser's own indicator, and an indicator that is usually absent is the easiest kind to counterfeit: there is nothing to compare it against. An origin that is always there, always in the same place, and outside the content area is a much poorer target.

This does not make the window unspoofable. It makes the spoof compete with a real control in a fixed position rather than filling a vacuum.
