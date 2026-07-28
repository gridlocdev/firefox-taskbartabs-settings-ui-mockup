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

### 4.7 Open links to this site here

| | |
|---|---|
| **Control** | Toggle |
| **Default** | Inherits §1.2 at the time the tab was added |
| **Disabled when** | §1.2 is off |

| State | Behaviour |
|---|---|
| **On** | Links to this tab's host that you open in Firefox come to this window instead of a new tab. |
| **Off** | Those links open as ordinary tabs. |
| **Disabled** | Shows *"Turn on 'Open links in their Taskbar Tab' in Taskbar Tabs settings to use this."* The stored value is kept. |

Matching is by host, so it covers the whole site, not just the configured address.

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

Opens the remove confirmation (§5.4).

Wording is the same for both kinds. What is removed is the Taskbar Tab: the site is not uninstalled, and it stays available in a normal tab, which is what the dialog already says.

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

**When the address resolves to a site that provides an app**, a choice appears between the Address and Name fields rather than the browser deciding silently:

| Option | Effect |
|---|---|
| **Add {name}, the app this site provides** (default) | Name is prefilled from the site and stays editable; the address becomes the app's start page, and the field description says so |
| **Add a shortcut to this page** | Name and address are both the person's, exactly as before |

**Duplicates** are caught on submit. If a Taskbar Tab already opens the resulting address, the field is marked invalid and shows *"{name} already opens this address. Open it from the list, or choose a different page."*

On success the tab is added to the top of the list and a toast confirms, offering **Open**.

![The Add a Taskbar Tab dialog](docs/screenshots/spec/5.1-add-dialog.png)

![The Add dialog offering the app or a shortcut](docs/screenshots/spec/5.1-add-app-choice.png)

*Shown for a site that provides an app. Choosing the app replaces the "opens at exactly this address" line, since the start page is the site's to set.*

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

Two kinds of entry share the list. One was typed in by hand; the other was declared by the site in a manifest, where the name, icon, start page and area belong to the developer.

| | Added by you | From the site |
|---|---|---|
| **Name** | Yours | The site's, replaceable with a local one (§4.1) |
| **Address it opens** | Any valid `http(s)` URL | The declared start page, editable within the app's area (§4.2) |
| **Area it covers** | Inferred: the host of the address | Declared by the site |
| **Icon** | Favicon | From the site |
| **Can change on its own** | No | Yes: the name, when the site is updated (§4.12) |

![A row showing the From the site badge alongside the others](docs/screenshots/spec/3.1-row-badges.png)

They share one list rather than two sections. Someone looking for Teams should not have to know how Teams arrived in order to find it. The difference is a property of a row, shown as a badge (§3.1), not an organising principle for the page.

**The badge is not a trust signal.** A manifest is asserted by whoever controls the origin, and Firefox verifies nothing beyond that, so **From the site** carries no checkmark, shield or "Verified".

### Why the name is editable and the address is not

The name is a private label with no security role, and renaming an installed app is the thing most often missing from equivalent features elsewhere, since installing one site across several profiles leaves several identically named icons. Locking it would buy nothing.

The address is different. A window that presents itself as an app, wears the site's icon, sits on the taskbar and holds the site's cookies and permissions is making a claim about what it is. Letting it be repointed anywhere is how that claim becomes a lie. So the address is held to the area the app declares, which is the site's own statement about its extent, not a restriction Firefox invented.

### Not built

- Two list sections, a Type column, or a provenance filter.
- A hidden advanced override for the address (§4.2).
- Editable icons, for either kind.
- A verified or trusted badge.
- Separate "Uninstall" vocabulary (§4.10).
