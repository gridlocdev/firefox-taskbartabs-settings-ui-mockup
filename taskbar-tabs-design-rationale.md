# Taskbar Tabs Management: Design Rationale

Firefox 143 gives you the function to change a site into a Taskbar Tab. This design adds the second half of that function. The [options specification](taskbar-tabs-options-specification.md) tells you what each control does. Thus this document gives only the reasons, and primarily the reasons for the less obvious decisions.

---

## The name of the feature

Two names are in use. The support documents say "web apps". The Firefox source code and the public Mozilla discussion say "Taskbar Tabs". This design uses Taskbar Tabs in all locations, in title case, as the name of the feature. A feature that uses two different names on one screen teaches the user nothing. There is one exception: the button in the address bar keeps the label **Add to taskbar**, because that is its true label in Firefox today.

## An item in the sidebar, not a card in a different page

The new Firefox Settings has a flat sidebar with icons and groups of items. Thus you do not have to put this feature into a category. To manage a list of installed items, you need more space than a settings card gives. Also, the feature is not a property of tab behaviour. Thus Taskbar Tabs has its own item directly below Tabs and browsing, where a person who looks for the feature goes through.

That layout has no breadcrumbs, and this design has none. Only the page for each app needs a control that goes back. That page keeps an **All Taskbar Tabs** button. This is better than a navigation pattern that the remainder of Settings does not use.

## Three settings, and no master switch

An early draft had a master on/off switch above the other settings. The switch made the page dim when it was off. This design removed the switch, because it hid too much consequence behind one control. The switch made a list of visible apps dim, and the shortcuts on the taskbar changed their behaviour without a message. Each other possible global setting is a property of an individual tab. Thus the card holds three checkboxes and no more.

The three controls are checkboxes and not toggles, because of the rule in Acorn. A toggle turns a feature on and off. A checkbox selects a configuration. The label shows the difference. "Taskbar Tabs" would be a toggle, but "Reopen Taskbar Tab windows when Firefox restarts" is clearly a configuration.

## The Restore defaults button hides itself

This pattern comes from `about:preferences#home`. There, the button is on the heading row of the section and stays invisible until a value is different from its default. That behaviour was a specific UX request during development. The reason in the code review was direct: if the button is always visible, you release a button that appears to do nothing when the user clicks it.

Our version has one difference. The Firefox version resets preferences outside its own section, and reviewers on that bug said that this is not clear. Our version resets only the three checkboxes directly below it.

## The name and the address share one screen

To change a name and to change an address are the same operation internally. Both operations make the shortcut file that Windows uses to start the app again. One screen thus gives one confirmation, one save operation, and one mental model. For this reason, **Rename…** goes to the settings page of the app and selects the name field. A dialog would be a second method to do one half of the operation.

This is also the only part of the interface that keeps changes until you push Save. Each other change is a preference change, but these two changes write a file to the disk.

## The save confirmation informs and does not warn

Nothing becomes lost or defective when Firefox makes a shortcut again. It is only possible that you must pin the app again. Thus the dialog uses the information style and not the warning style. An earlier version put the URL in a box with a border. There, the URL looked the same as a text input in a dialog that is full of text inputs. The URL is now bold body text, as the Acorn documentation specifies. The user cannot mistake bold body text for a field to complete.

## Containers stay editable

At one time, containers were read-only, because a change of container has large effects. That decision was too careful. Users have correct reasons to move a tab to a different container. Also, if you remove the option, you do not remove the effects: the user deletes the tab and adds it again. The confirmation says that you *can* be signed out, and does not say that you will be signed out. The result depends on the contents of the container that you move to.

## Two types of entry, one list

A site that supplies a manifest declares its own name, icon, start page, and area. These values are the identity of the app, not a preference of the user. This is a true difference from a Taskbar Tab where a person typed an address. The page must show the difference. Two rows that look the same but behave differently after a click are the most severe usability failure that is possible here.

The interface shows the difference as a badge, **From the site** or **Added by you**. The badge joins the pinned badge and the container badge that the row already has. The obvious alternative was two sections in the list, and this design refused it. A person who looks for Teams must not have to know how Teams came into the list. Both states have a badge, and not only one state. This follows the example of **Pinned to taskbar** and **Not pinned**. A user cannot tell the difference between a badge that is not there and a badge that the user did not see.

"Installed app" and "Shortcut" would be more familiar. These terms come from Chromium and from telephones, but they lost. Firefox installs both types in the same manner, and writes the same shortcut file in the same manner. Thus the word "installed" for one of the two types states a mechanical difference that does not exist. This is the same type of untrue statement as the unpin toggle below.

The badge is intentionally not a checkmark and not a shield. The party that controls the origin asserts the manifest, and Firefox does no more verification. Thus a badge that looks like a trust signal would be a security statement that the browser cannot support.

## The name is yours, but the address belongs to the site

The first instinct is to lock both values, because the developer defined them. This design locks neither value, and constrains only one.

The name stays editable. It is a private label. Firefox never transmits it, never uses it in a decision about an origin, and never uses it as a security boundary. Also, users of Edge have asked for the function to rename an installed app for many years. If you install one site in more than one profile, you get more than one icon with the same name, and this is the manifest condition specifically. To lock the name would make a dead end in the only scenario that needs the function, and would give no security advantage. To keep the claim of the developer, the interface shows the name from the site permanently below the field, with a reset control adjacent to it. It does not remove the name of the user.

The address is the part that carries the identity. A window has the icon of the site, is on the taskbar, and holds the cookies and permissions of the site. Thus the window makes a statement about what it is. If you can point the window to a different location, that statement becomes untrue. Thus the address must stay in the area that the app declares. That area is the statement of the site about its own extent, and Firefox did not invent it. In that area, the address is fully editable. Thus the app can open at the calendar and not at the inbox.

The field is constrained, but not disabled. A disabled field answers only "you cannot". The user must then guess if the field is defective, unsupported, or forbidden. A field with validation gives the rule at the moment that the rule is important, and gives the alternative. The alternative is a different object with an honest label: to add a Taskbar Tab. An advanced override would add only the condition that this design must prevent.

## The rename notice is an approval mechanism, and Chrome built the same mechanism

A manifest makes it possible for an app to change its own name. If a taskbar icon can change from **Outlook** to a different name because a developer edited a JSON file, the browser must have a position on that behaviour. That position cannot be silence.

Chrome states the problem correctly: *"Because Web Apps lack a central authority like Google Play to review updates, these modifications must be presented clearly to users for confirmation."* There is no reviewer. Thus the person who operates the browser is the reviewer, and the only true task of the browser is to show the change to that person honestly. The answer from Chrome is in [Chrome 144](https://developer.chrome.com/blog/improvements-to-web-app-updates). It divides the manifest into two parts. Icons and names are *"security sensitive members"* and Chrome holds them separately. Each other member applies immediately and invisibly. A change of identity waits behind a **Review app update** item in the menu of the app until the person opens it. The version before that one was worse in the important manner: *"a disruptive dialog requiring them to either uninstall the app or immediately accept icon and name changes"*. That modal asks for a decision before you can do the task that you opened the app to do.

§4.12 is that mechanism. The design keeps identity separate from configuration. Configuration applies without a message. A change of name appears in the location where the person already goes to manage the app, and does not interrupt the person at start-up. Two browsers found the same division independently, and that is the useful part. It shows that the line between identity and configuration is real, and not a Firefox peculiarity.

Our design has one difference, and the local override makes that difference possible. Chrome holds the change and asks the person to accept it. This design adopts the new name and offers **Keep calling it {old}**. To refuse here is not to stop an update. It is to write a label with the same mechanism as a manual rename. Thus no app keeps an old identity that the browser must compare with the live manifest. It also means that the notice goes only to persons who never gave a preference. Firefox never replaces a name that a person set, and never asks about it.

The other two rules from Chrome apply here without disagreement. Chrome applies icon changes below a difference of 10% of the pixels automatically, at a maximum of one change each day. That is a sensible answer to a question that this design does not have, because icons are not editable here in either direction. That threshold belongs with the icon pipeline, and not with the name. Also, a change by the developer to `start_url` or to `scope` applies automatically. This agrees with Chrome, which treats these members as configuration. The useful consequence is that the constraint on the address field moves with the app. It does not stay at the value that the manifest had on the day of the installation.

Both designs have one gap. If nobody looks for a change, nobody sees it. The change from Chrome is in a menu, and our change is on a page that you must open.

## The window is the location of the true security

Each other part of this design is a settings page, and an attacker does not go through a settings page. The constraint on the address prevents a change of the address of a Taskbar Tab through Settings, and that is the expensive attack. The inexpensive attack is different: the app moves off its own origin immediately after it opens, in a window that has no address bar, has the icon of the site, and holds its cookies.

Thus the window always shows the origin, in true browser chrome. The appearance of the origin changes when the app goes out of the area that it declares. Chromium shows nothing while the app stays in its area, and raises a small URL bar when the app goes out of the area. That behaviour is better than nothing, and worse than this design, for one reason. A page in standalone mode can draw a good copy of the indicator of the browser. An indicator that is usually absent is the easiest type to copy, because there is nothing on the screen to compare the copy with. A control that is always present, always in the same location, and outside the content area is a much more difficult target. This does not make a false indicator impossible. It makes the false indicator compete with a true indicator, and not fill an empty space.

The same location carries the condition for an insecure connection. An `http://` shortcut to a device on your own network is a correct thing to want, and `homeassistant.local:8123` is the reason that this design permits it. Thus the answer is not to refuse the address, but to give the information in the correct location. This is also the reason that the list has no badge for this condition. You learn nothing when the interface tells you that an address that you typed intentionally is the address that you typed. A manifest is different. Firefox never trusts a manifest over plain HTTP, because an identity that each person on the network can edit is not an identity.

## The Add dialog makes shortcuts, and only shortcuts

Chromium installs an app from a page that you already have open. The browser already has the manifest, the origin is already in your history, and you decide to install the app after you decide to visit the site. Our Add dialog did the opposite. You type an address, and Settings must then get the manifest of that origin to know what to offer. This occurs before you agree to anything, and possibly for an address that you copied from a different person.

That request is a request that the person did not ask for. It goes to a server that the person can be unfamiliar with, and it comes from a page where nobody expects network activity. The correct answer is not to make the request quieter. The correct answer is that the address bar is already the correct location, because the browser loads the page there by definition. Thus the dialog makes shortcuts, and the address bar installs apps. If a shortcut points at a site that supplies an app, the settings page of that shortcut offers the change. That offer also needs no speculative request.

Each entry point now does one task, and neither entry point must guess.

## Containers make one app into two, and each function downstream must agree

Chromium keys an app on its manifest `id` in a profile. Thus one app is one entry. The isolation unit in Firefox is inside the profile. Thus two containers can run one app, and the manifest `id` alone is no longer an identity.

The first version of this design had a true defect. The duplicate check compared the address and ignored the container. Thus Firefox refused Outlook in Work and Outlook in Personal. That defect stopped the one capability that this design has and Chromium does not have, in the exact condition that containers exist for.

The key is the manifest and the container together. That key causes three smaller answers. First, the two copies are different by name, and the second copy must have a different name. The taskbar shows neither the container nor this settings page, and two identical icons where one icon is your work account are a true hazard. Second, only one copy captures links, and you select that copy explicitly. A link has one destination. The alternatives are a dialog in the path of each click, or a rule about recency that nobody can predict. Third, both copies can start at sign-in, because two windows at sign-in is a sensible thing to want. Two destinations for one link is not.

## The interface shows permissions but does not edit them

This is the most important honesty decision on the page. Firefox gives permissions to a site, and not to a Taskbar Tab. Thus, if you block the camera for `outlook.office.com`, you block it in the app and in each usual tab. Dropdowns here would operate correctly, but they would also teach the user something untrue about Firefox. Thus the page shows the current state and links to the site permissions, where the interface shows the change at its true extent.

## The taskbar pin is the constraint that controlled the design

Windows controls the taskbar. Microsoft locked programmatic pin operations many years ago. Thus a browser can ask Windows to add an icon, but it cannot remove one. The Microsoft instructions for Edge also tell the user to click the icon with the right button and select Unpin. Early drafts had a "Pinned to taskbar" toggle and a bulk "Unpin from taskbar" button. Both controls were untrue. A switch that the operating system ignores is worse than no switch.

Thus there is no pin toggle, no unpin action in any location, and no bulk unpin control. The pinned badge is a label and not a control. The two operations that can remove a pin are the removal of a pinned tab and a new build of its shortcut. Both operations say so clearly.

## What this design does not include, and why

- **An on/off switch for each app.** To remove a tab is the honest version of a switch that turns a tab off.
- **A control to change the icon.** Icons come from the site. An override causes its own storage and fallback questions.
- **Editable permissions for each app.** See above.
- **An advanced override for the address of a site app.** You can satisfy each correct need if you edit the address in the area of the app or add a separate Taskbar Tab. An override would add only the incorrect condition. Also, hidden advanced flows are the most dangerous for the persons who are the most easy to deceive.
- **File handlers and protocol handlers.** Chromium writes the members that a manifest declares into the Windows registry, and removes them at uninstall. To claim `.pdf` or a URL scheme is an assertion that Firefox must truly make. To offer a control that the operating system ignores is the unpin toggle again. This document records the liability and does not hide it: if handlers come later, the removal text that promises that only a shortcut goes away becomes untrue.
- **A badge in the list for an insecure origin.** The window carries that information. A list of items that you configured intentionally is a poor location for a message about an address that you typed.
- **Two sections in the list, a Type column, or a filter for the source of an app.** These cost recall and permanent space, and they group the list by a fact that is important on two screens.
- **A separate "Uninstall" vocabulary.** Firefox uninstalls nothing more today. Examine this again if manifest apps get a true installed state, for example handler registrations or their own storage bucket.
- **Mobile support.** Android and iOS install sites through fully different operating system mechanisms. Thus none of this design applies there.

## Tokens, and not new components

The colour, the type, the space, and the components come from the published Acorn tokens. Where a component was not available, this design used a token and did not invent a component. For example, the emphasised URL in the save dialog started as a custom monospace code block and became documented bold body text. Two accessibility decisions have the same reason. Hidden controls use `visibility: hidden`. Thus they keep their space and stay out of the tab order. Also, colour is never the only signal: a container badge has a coloured dot and the name of the container.

## A comparison with Chrome and Edge

Chromium already has these functions: open in a window, run at login, link capture for each app, access to permissions, and deletion of data at uninstall. This design has the same functions. Chromium does not have these functions: rename after installation, an editable address, containers, search, sort, and bulk removal. The rename function is important. It has been an open Edge feature request for many years, because installation of the same site in more than one profile gives more than one app with the same name. This design does not include the file handling toggle from Chromium or its enterprise policy surfaces.

## Open questions

1. **Area against start page, for shortcuts.** Manifest apps solve this problem without work: the declared area is the capture area, and the start page is separate. A shortcut declares nothing. Thus its address has two functions, and its capture falls back to the full host, which is the more coarse rule. It is not decided if a shortcut can declare its own area, or if such a shortcut is only an app with more steps.
2. **If a pin stays after Firefox makes a shortcut again.** This design warns that the pin can go away. A test on true hardware must show what Windows does.
3. **macOS.** The Dock needs a true `.app` bundle for each app. That is a different type of problem, and it is the primary reason that this design starts with Windows.
4. **When Firefox starts to read manifests.** Taskbar Tabs today takes its name and icon from the page. Thus the second type of entry does not exist yet. This work is intentionally in front of that change. The interface for an identity that a developer owns must be correct before the first app comes with such an identity, and not after. The schedule is not decided, and the sequence of the manifest members is not decided.
5. **The text of the badges.** "From the site" and "Added by you" won because they are honest. "Installed app" and "Shortcut" would win because they are familiar. The two arguments are near to each other. It is better to test them than to discuss them.
6. **If a manifest update must build the shortcut again.** A new build can cause the loss of the pin. Thus an edit by a developer can remove an app from the taskbar without a message. It can be safer to adopt the new name in the app and to delay the new build. This question needs the same test on true hardware as question 2.
7. **If the permanent origin costs too much of the app appearance.** The purpose of a Taskbar Tab is that it does not look like a browser. A permanent origin in the title bar is a permanent message that it is a browser. The trade is intentional, but only a true build can decide it.
8. **If one holder of the link capture is the correct rule.** Test this when a person truly operates two work accounts at the same time. The rule won because it is predictable, but predictable is not the same as convenient.
