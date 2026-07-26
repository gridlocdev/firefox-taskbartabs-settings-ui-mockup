# Managing Taskbar Tabs: Design Rationale

Firefox 143 shipped the ability to turn a site into a Taskbar Tab, and this design adds the half that came after it. What every control does is in the [options specification](taskbar-tabs-options-specification.md), so this document covers only the reasoning, and mostly the places where we chose the less obvious option.

---

## The name

Both names are in circulation: the support docs say "web apps", while Firefox's own source and Mozilla's public discussion say "Taskbar Tabs". We use Taskbar Tabs throughout, in title case, as a proper feature name, because a feature that calls itself two different things on one screen teaches people nothing. The address bar button is the one exception, still labelled **Add to taskbar**, since that is its real label in Firefox today.

## Its own sidebar item, not a card inside another page

Firefox's redesigned Settings has a flat, icon-led sidebar with grouped entries, so the question is no longer which category to bury this in. Managing a list of installed things needs more room than a settings card gives, and the feature is not a property of tab behaviour, so Taskbar Tabs gets its own entry directly below Tabs and browsing, where someone looking for it will pass through anyway.

That layout has no breadcrumbs, so this design has none either. The one place that still needs a way back is the per-app page, which keeps an explicit **All Taskbar Tabs** button rather than reintroducing a navigation pattern the rest of Settings dropped.

## Three settings, and no master switch

An early draft put a master on/off switch above the others, dimming the page when off. It went because it hid too much consequence behind one control: a switch that greys out a list of apps you can still see, while shortcuts already on your taskbar quietly change behaviour. Everything else that could have been global turned out to belong to individual tabs instead, so the card holds three checkboxes and nothing more.

They are checkboxes rather than toggles because of Acorn's rule that toggles turn a feature on and off while checkboxes pick a configuration. The label gives it away. "Taskbar Tabs" would be a toggle, but "Reopen Taskbar Tab windows when Firefox restarts" is plainly a configuration.

## Restore defaults hides itself

Copied from `about:preferences#home`, where the button sits on the section heading row and stays invisible until something differs from its default. That was a specific UX request during development, and the reasoning in code review was blunt: otherwise you ship a button that appears to do nothing when clicked. Ours differs in one way. Firefox's version resets preferences beyond the section it appears in, which reviewers on that bug flagged as confusing, so ours resets only the three checkboxes directly beneath it.

## Rename and address share one screen

Renaming and re-addressing are the same operation underneath, since both rebuild the shortcut file Windows uses to launch the app. Giving them one screen means one confirmation, one save and one mental model, which is why **Rename…** jumps to the app's settings page with the name field selected rather than opening a dialog that would have been a second way to do half the job.

This is also the only part of the interface that holds changes until you press Save. Everywhere else a change is a preference flip, while these two rewrite a file on disk.

## The save confirmation informs rather than warns

Nothing is lost or broken when a shortcut is rebuilt, and you may simply have to pin the app again, so the dialog takes the information style instead of a warning. An earlier version put the URL in a bordered box, where it looked exactly like a text input sitting in a dialog full of text inputs. It is now Acorn's documented bold body text, which cannot be mistaken for something you are supposed to fill in.

## Containers stay editable

Containers were read-only at one point, on the grounds that switching one is disruptive. That was over-cautious, because people have legitimate reasons to move a tab between containers, and taking the option away does not remove the disruption; it just makes them delete and re-add the tab. The confirmation says you *may* be signed out rather than promising that you will, since the outcome depends on what is already in the container you are moving to.

## Permissions are shown, not edited

This is the most important honesty decision on the page. Firefox grants permissions to a site rather than to a Taskbar Tab, so blocking the camera for `outlook.office.com` blocks it in the app and in every ordinary tab. Editable dropdowns here would have worked perfectly well and would also have taught people something false about how Firefox works, so the page shows the current state and links out to site permissions, where the change is presented at its true scope.

## Pinning is the constraint that shaped everything

Windows controls the taskbar. Programmatic pinning was locked down years ago, so a browser can ask to add an icon but cannot take one away, and even Microsoft's guidance for Edge tells people to right-click the icon and choose Unpin. Early drafts had a "Pinned to taskbar" toggle and a bulk "Unpin from taskbar" button, and both were fiction. Flipping a switch that the operating system will ignore is worse than not offering the switch at all.

So there is no pin toggle, no unpin action anywhere, and no bulk unpin. The pinned badge is a label rather than a control, and the two moments where a pin may not survive, removing a pinned tab and rebuilding its shortcut, both say so plainly.

## What we deliberately didn't build

- **A per-app on/off switch.** Removing a tab is the honest version of turning one off.
- **Change icon.** Icons come from the site, and overriding them brings its own storage and fallback questions.
- **Editable per-app permissions.** Covered above.
- **Anything mobile.** Android and iOS install sites through entirely different OS mechanisms, so none of this design applies there.

## Tokens rather than invented components

Colour, type, space and components all come from Acorn's published tokens. Where something was missing we reached for a token instead of inventing a component: the emphasised URL in the save dialog began as a custom monospace code block and ended up as documented bold body text. Two accessibility choices are worth naming for the same reason. Hidden controls use `visibility: hidden`, so they keep their space and stay out of the tab order, and colour is never the only signal, with container badges pairing a coloured dot with the container's name.

## Next to Chrome and Edge

Chromium already offers open in window, run at login, per-app link capture, permissions access and clear-data-on-uninstall, and this design matches those. Renaming after install, editing the address, containers, search, sort and bulk removal are things it does not have. Renaming in particular has been an open Edge feature request for years, since installing the same site across several profiles leaves you with several identically named apps. We skip Chromium's file-handling toggle and its enterprise policy surfaces.

## Open questions

1. **Scope versus start page.** The address does double duty today: where the app opens, and by host, which links it captures. Firefox may eventually want those separate, with a start page of `/mail/inbox` and a capture scope covering the whole domain.
2. **Whether a pin survives a rebuilt shortcut.** We warn that it may not. What Windows actually does needs testing on real hardware.
3. **macOS.** The Dock needs a real `.app` bundle per app, which is a different shape of problem and the main reason this is Windows-first.
