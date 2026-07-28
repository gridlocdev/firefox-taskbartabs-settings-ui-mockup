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

## Two kinds of entry, one list

A site that ships a manifest declares its own name, icon, start page and scope, and those are the app's identity rather than the user's preference. That is a genuine difference from a Taskbar Tab someone typed an address into, and the page has to show it, because rows that look identical and behave differently when clicked is the sharpest usability failure available here.

It is shown as a badge, **From the site** or **Added by you**, joining the pinned and container labels the row already carries. Splitting the list into two sections was the obvious alternative and it lost: someone looking for Teams should not have to know how Teams arrived in order to find it. Both states get a badge rather than only one, following the precedent set by **Pinned to taskbar** / **Not pinned**, since a missing badge cannot be told apart from a badge you failed to notice.

"Installed app" and "Shortcut" would have been more familiar, borrowed from Chromium and from phones, and they lost anyway. Both kinds install identically, the same shortcut file written the same way, so calling one of them installed asserts a mechanical difference that does not exist. That is the same class of fiction as the unpin toggle below.

The badge is deliberately not a checkmark or a shield. A manifest is asserted by whoever controls the origin and Firefox verifies nothing beyond that, so a trust-shaped badge would be a security claim the browser cannot back.

## The name is yours, the address is the site's

The instinct is to lock both, since the developer defined them. We lock neither, and constrain only one.

The name stays editable. It is a private label: never transmitted, never used in an origin decision, never a security boundary. Meanwhile renaming an installed app is exactly the thing Edge users have been asking for for years, because installing one site across several profiles leaves several identically named icons, and that is the manifest case specifically. Locking it would create a dead end in the only scenario that needs it, for no security gain. The developer's claim survives by being shown, since the site's name sits under the field permanently with a reset beside it, rather than by suppressing the user's.

The address is the part that carries the identity. A window wearing the site's icon, sitting on the taskbar and holding the site's cookies and permissions is making a claim about what it is, and repointing it anywhere is how that claim becomes a lie. So it is held to the app's declared scope, which is the site's own statement about its extent rather than something Firefox invented. Within that area it is fully editable, so opening at the calendar instead of the inbox just works.

Constrained, not disabled. A disabled field answers "you can't" and leaves you guessing whether it is broken, unsupported or forbidden. A validated one names the rule at the moment it matters and hands over the alternative, which is the whole point of an escape hatch that is a different object honestly labelled, add a Taskbar Tab, rather than an advanced override that would add nothing except the case we are trying to prevent.

## The rename notice is an approval mechanism, and Chrome built the same one

The thing a manifest makes possible is an app renaming itself. If a taskbar icon can go from **Outlook** to something else because a developer edited a JSON file, the browser has to have a position on that, and the position cannot be silence.

Chrome states the problem exactly: *"Because Web Apps lack a central authority like Google Play to review updates, these modifications must be presented clearly to users for confirmation."* There is no reviewer, so the person running the browser is the reviewer, and the browser's only real job is to put the change in front of them honestly. Chrome's answer, shipped in [Chrome 144](https://developer.chrome.com/blog/improvements-to-web-app-updates), splits the manifest in two: icons and names are *"security sensitive members"* held separately, everything else applies immediately and invisibly, and an identity change waits behind a **Review app update** entry on the app's own menu until the person gets to it. The version before that was worse in the way that matters: *"a disruptive dialog requiring them to either uninstall the app or immediately accept icon and name changes"*, which is the modal that demands a verdict before you can do the thing you opened the app to do.

§4.12 is that mechanism. Identity is separated from configuration, configuration applies without comment, and a name change surfaces where someone already goes to manage the app rather than ambushing them at launch. Two browsers reaching the same split independently is the useful part: it suggests the line between identity and configuration is real rather than a Firefox idiosyncrasy.

Ours differs in one respect, and the local override is what makes it affordable. Chrome holds the change pending and asks the person to accept it. We adopt the new name and offer **Keep calling it {old}**, because declining here is not blocking an update, it is writing a label using the same mechanism as a manual rename, so no app is left holding a stale identity the browser has to keep reconciling against the live manifest. It also means the notice only ever reaches people who never expressed a preference, since a name someone set is never overwritten and never asked about.

Chrome's other two rules transfer without argument. Icon changes under a 10% pixel difference apply automatically, throttled to once a day, which is a sensible answer to a question this design does not have, because icons are not editable here in either direction; that threshold belongs with the icon pipeline rather than with the name. And a developer's change to `start_url` or `scope` applies automatically, matching Chrome's treatment of them as configuration, with the useful consequence that the constraint on the address field moves with the app instead of pinning it to whatever the manifest said on the day it was installed.

Both designs share one gap. A change nobody goes looking for is never seen: Chrome's sits in a menu, ours sits on a page you have to visit.

## The window is where the security actually lives

Everything else in this design is a settings page, and a settings page is not what an attacker goes through. The address constraint stops a Taskbar Tab being repointed through Settings, which is the expensive attack. The cheap one is the app navigating itself off its own origin the moment it opens, in a window with no address bar, wearing the site's icon and holding its cookies.

So the origin is always shown, in real chrome, and it changes appearance when the app leaves the area it declares. Chromium shows nothing in scope and raises a mini URL bar on leaving it, which is better than nothing and worse than this for a specific reason: a page in standalone mode can draw a passable imitation of the browser's own indicator, and an indicator that is normally absent is the easiest kind to fake, because there is nothing on screen to compare the fake against. A control that is always present, always in the same place, and outside the content area is a much poorer target. It does not make spoofing impossible. It makes the spoof compete with a real thing instead of filling a vacuum.

The same slot carries the insecure case. An `http://` shortcut to a device on your own network is a legitimate thing to want, and `homeassistant.local:8123` is why the design allows it, so the answer is not to refuse it but to say so where it matters. That is also why the list does not badge it: nothing is learned from being told an address you typed on purpose is the address you typed. A manifest is different, and is never trusted over plain HTTP at all, because an identity anyone on the network can edit is not an identity.

## The Add dialog makes shortcuts, and only shortcuts

Chromium installs from a page you already have open. The manifest is already fetched, the origin is already in your history, and the decision to install comes after the decision to visit. Our Add dialog inverted that: type an address, and Settings has to go and fetch that origin's manifest to know what to offer, before you have committed to anything and possibly for an address you pasted from someone else.

That is a request the person did not ask for, to a server they may not know, from a page where nobody expects network activity. The fix is not to make the fetch quieter. It is to notice that the address bar is already the right place, because the page is loaded there by definition. So the dialog makes shortcuts, the address bar installs apps, and a shortcut that turns out to point at a site providing an app is offered the switch on its own settings page, which needs no speculative fetch either.

Each entry point now does one thing and none of them has to guess.

## Containers make one app into two, and everything downstream has to cope

Chromium keys an app on its manifest `id` inside a profile, so one app is one entry. Firefox's isolation unit sits inside the profile, which means two containers can run one app, and the manifest `id` alone stops being an identity.

The first version of this shipped a real bug: duplicate detection compared the address and ignored the container, so adding Outlook in Work and again in Personal was refused. That broke the one capability this design has that Chromium does not, in exactly the case containers exist for.

The key is the manifest plus the container, and that forces three smaller answers. Two copies get told apart by name, which the second copy is required to make distinct, because the taskbar shows neither the container nor this settings page and two identical icons where one is your work account is a genuine hazard. Only one copy captures links, chosen explicitly, because a link has one destination and the alternative is either a dialog in the path of every click or a rule about recency that nobody can predict. Both may start at sign-in, because wanting two windows is coherent in a way that wanting a link to go to two places is not.

## Permissions are shown, not edited

This is the most important honesty decision on the page. Firefox grants permissions to a site rather than to a Taskbar Tab, so blocking the camera for `outlook.office.com` blocks it in the app and in every ordinary tab. Editable dropdowns here would have worked perfectly well and would also have taught people something false about how Firefox works, so the page shows the current state and links out to site permissions, where the change is presented at its true scope.

## Pinning is the constraint that shaped everything

Windows controls the taskbar. Programmatic pinning was locked down years ago, so a browser can ask to add an icon but cannot take one away, and even Microsoft's guidance for Edge tells people to right-click the icon and choose Unpin. Early drafts had a "Pinned to taskbar" toggle and a bulk "Unpin from taskbar" button, and both were fiction. Flipping a switch that the operating system will ignore is worse than not offering the switch at all.

So there is no pin toggle, no unpin action anywhere, and no bulk unpin. The pinned badge is a label rather than a control, and the two moments where a pin may not survive, removing a pinned tab and rebuilding its shortcut, both say so plainly.

## What we deliberately didn't build

- **A per-app on/off switch.** Removing a tab is the honest version of turning one off.
- **Change icon.** Icons come from the site, and overriding them brings its own storage and fallback questions.
- **Editable per-app permissions.** Covered above.
- **An advanced override for a site app's address.** Every legitimate need is met by editing within the app's area or adding a separate Taskbar Tab. An override would add only the illegitimate case, and hidden advanced flows are worst for exactly the people most likely to be talked into using them.
- **File and protocol handlers.** Chromium registers what a manifest declares into the Windows registry and unregisters it on uninstall. Claiming `.pdf` or a URL scheme is an assertion Firefox would have to actually make, and offering a control the operating system ignores is the unpin toggle over again. The liability is recorded rather than hidden: if handlers ever land, the removal copy promising that only a shortcut goes away stops being true.
- **An insecure-origin badge in the list.** The window carries it. A list of things you configured on purpose is a poor place to be told about the address you typed.
- **Two list sections, a Type column or a provenance filter.** Costs recall and permanent space to organise by a fact that matters on two screens.
- **Separate "Uninstall" vocabulary.** Nothing extra is uninstalled today. Worth revisiting if manifest apps ever gain real installed state, like handler registrations or their own storage bucket.
- **Anything mobile.** Android and iOS install sites through entirely different OS mechanisms, so none of this design applies there.

## Tokens rather than invented components

Colour, type, space and components all come from Acorn's published tokens. Where something was missing we reached for a token instead of inventing a component: the emphasised URL in the save dialog began as a custom monospace code block and ended up as documented bold body text. Two accessibility choices are worth naming for the same reason. Hidden controls use `visibility: hidden`, so they keep their space and stay out of the tab order, and colour is never the only signal, with container badges pairing a coloured dot with the container's name.

## Next to Chrome and Edge

Chromium already offers open in window, run at login, per-app link capture, permissions access and clear-data-on-uninstall, and this design matches those. Renaming after install, editing the address, containers, search, sort and bulk removal are things it does not have. Renaming in particular has been an open Edge feature request for years, since installing the same site across several profiles leaves you with several identically named apps. We skip Chromium's file-handling toggle and its enterprise policy surfaces.

## Open questions

1. **Scope versus start page, for shortcuts.** Manifest apps resolved this for free: the declared scope is the capture area and the start page is separate. A shortcut declares nothing, so its address still does double duty and its capture falls back to the whole host, which is the coarser rule. Whether a shortcut should be able to state its own area, or whether that is just an app with extra steps, is unsettled.
2. **Whether a pin survives a rebuilt shortcut.** We warn that it may not. What Windows actually does needs testing on real hardware.
3. **macOS.** The Dock needs a real `.app` bundle per app, which is a different shape of problem and the main reason this is Windows-first.
4. **When Firefox starts reading manifests.** Today's Taskbar Tabs takes its name and icon from the page, so the second kind of entry does not exist yet. The provenance work is deliberately ahead of that: the interface for developer-owned identity is the part that has to be right before the first app arrives carrying one, not after. What is unresolved is the timing, and which manifest members land first.
5. **Badge wording.** "From the site" and "Added by you" won on honesty; "Installed app" and "Shortcut" would win on familiarity. Close enough to be worth testing rather than arguing.
6. **Whether a manifest update should rebuild the shortcut.** Rebuilding may cost the pin, so a developer's edit could silently unpin an app. Adopting the new name in-app and deferring the rebuild may be the safer trade, and it needs the same real-hardware testing question 2 does.
7. **Whether the always-present origin costs too much of the app feel.** The whole point of a Taskbar Tab is that it does not look like a browser, and a permanent origin in the title bar is a permanent reminder that it is one. The trade is deliberate, but it is the kind of thing that only a real build settles.
8. **Whether a single link-capture holder is the right rule** once someone actually runs two work accounts side by side. It is the predictable answer, which is why it won, but predictable is not the same as convenient.
