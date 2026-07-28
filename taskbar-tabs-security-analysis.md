# Security and feature gaps, compared with Chromium

This document reviews the design in its current state against the equivalent Chromium functions for installed web apps. It gives what is absent, what is incorrect, and what is intentionally different and must stay different.

**Summary:** the identity work in [the source analysis](taskbar-tabs-provenance-analysis.md) is correct, but alone it protects the wrong end of the problem. An attacker has the least need of the address field in Settings. The unprotected surface is the window at run time, and the design says nothing about it.

**Status:** the design has answers for the ten findings. Two findings were defects, and [mockup.html](mockup.html) has the corrections. The [specification](taskbar-tabs-options-specification.md) holds each other decision. §11 records the result of each finding. The sections below stay as the reasoning that made those decisions. They use the present tense of the review.

---

## 1. The window is not defined, and it is the location of the security

Each document here describes a list, a sub-page, and some dialogs. No document describes the object that these settings make: a browser window with less chrome, which holds the cookies and permissions of a site, has the icon of that site, and starts from the taskbar.

That window has no address bar. If you remove the address bar, you remove the primary anti-phishing control in the browser. The design has no position on the replacement.

The position of Chromium is explicit. When a standalone app window goes to a different origin, Chromium shows a small URL bar with the new origin. It keeps the standalone display mode, but it makes the boundary visible. The manifest specification asks for exactly this behaviour: a prominent UI element that shows the origin as a minimum, and that is visually different from the state in the area, so that the user sees the change clearly.

**What needs a decision:**

| Question | The Chromium answer | Our answer |
|---|---|---|
| What identifies the origin while the app is in its area? | An origin chip in the title bar, and "App info" | Not defined |
| What occurs at a navigation out of the area? | A small URL bar shows the new origin | Not defined |
| Can the page make a copy of that indicator? | Partly, and this is a known open issue | Not defined |
| Does an insecure origin look different? | Yes, with the standard insecure indicator | Not defined |

The problem of the false indicator needs a statement, and this design must not take it silently from Chromium. [w3c/manifest#747](https://github.com/w3c/manifest/issues/747) records it: a page in standalone mode can draw a good copy of the off-scope indicator of the browser. Thus the indicator is a hint and not a guarantee. Firefox has an advantage here. It can put the indicator in true window chrome, and the content cannot draw over it. That is a true design opportunity, and not a gap to copy.

This is the largest omission. The area constraint on the address field (§4.2) prevents a user or a support fraud from a change to the address of an app in Settings. It does nothing about an app that goes to a different location immediately after it opens. That attack is less expensive, and it does not use Settings.

---

## 2. Manifest apps need a secure context, and no rule enforces it

Chromium does not install a web app from an origin that is not secure. It uses HTTPS or `localhost`, or it installs nothing. This rule is not incidental. An attacker can change a manifest that comes over plain HTTP while it is in transit. The identity of an app is exactly the value that a network attacker must not edit.

The prototype accepts `http://` for both types of entry. A test confirms this: the Add dialog accepts `http://insecure.example/login` with no message.

The rule must be divided in the same manner as in Chromium:

- **From the site**: needs a secure context, because the identity claim has value only when the transport is authenticated.
- **Added by you**: `http://` is permitted, because a shortcut to a local device is a true use case. The seed data already uses `http://homeassistant.local:8123`. But the window must then show the insecure indicator, and this returns to §1.

Note the conflict in the current seed data. Home Assistant is a plain-HTTP app that many persons truly operate, and it would be incorrect to refuse a shortcut to it. This difference solves the conflict cleanly, and that result supports the decision to have two types of entry.

---

## 3. Link capture uses the host, but Chromium uses the declared area

§4.7 compares by host, and the specification says so clearly: "Matching is by host, so it covers the whole site, not just the configured address."

Chromium captures each navigation that is in the area that the manifest declares.

A comparison by host is too wide on a shared host. An app that a person installs from `someuser.github.io` captures each link to `github.io`. An app on `tenant.atlassian.net` captures the host, and that result is correct. But `sharepoint.com`, or each other host with more than one tenant, is not correct. The failure is as follows: a link to the content of a different person opens in an app window that has the name of one tenant and holds the cookies of that tenant.

For manifest apps, the correction has no cost, because the area is already declared and the address field already parses it. For manual shortcuts, no area is declared. Thus a comparison by host is the only available option, and the honest statement is that manual shortcuts capture more coarsely than apps.

The rationale already records this as an open question about the difference between a start page and a capture area. This document must record it again as a security question, and not as a usability question.

---

## 4. `scope_extensions` would break the address field

Chromium has `scope_extensions`, which lets an app claim more origins. Each of these origins must agree, and it agrees when it holds `/.well-known/web-app-origin-association` with the name of the app. Neither party can declare the relation alone. The member exists because true apps use more than one origin. Without it, these navigations start the off-scope bar and lose the link capture.

Our `inScope` check compares one origin. An app that correctly uses `example.com` and `example.co.uk` gets a refusal from the address field for its own second origin. The error message then tells the user to add a separate Taskbar Tab.

This design does not have to implement the scope extensions. But it must not put the assumption of one origin into the code. The check is the part that a person must write again, and not extend. Thus the design must treat the area of the app as a set of matchers with a length of one today.

---

## 5. Containers and manifest identity collide, and this problem has no answer

There is no precedent to copy for this part. The isolation unit of Chromium is a profile, and the isolation unit of Firefox is a container in a profile.

Chromium keys an installed app on its manifest `id` in a profile. Thus one app is one entry with one set of registrations in the operating system. If two containers run the same app in one Firefox profile, two entries share one manifest `id`. Then several questions have no answer:

| Collision | Consequence |
|---|---|
| Link capture | A captured link matches two windows. Which container gets the link? |
| Run at sign-in | Both apps start, or one app starts. Not defined. |
| File handlers and protocol handlers | The operating system registration is one ProgID for each app. Two containers cannot both own `.pdf`. |
| Shortcut identity | Two shortcuts have the same manifest name and the same icon but different cookie jars. Nothing on the taskbar shows the difference. |

The last collision is a usability problem that becomes a security problem. Two identical icons are on the taskbar, one icon is your work account, and you cannot see which icon you clicked.

As a minimum, the design must state that the identity key is (manifest `id`, container) and not the manifest `id` alone. It must also state that an app with more than one container needs a visible difference. The container badge is already in the list. But it is not on the taskbar, in Alt+Tab, or in the Start menu, and that is where the confusion occurs. The local rename control is the available answer. The design can also ask for a name when a person adds a second copy of the same app in a different container.

---

## 6. The Add dialog makes a request that Chromium never makes

Chromium installs an app from a page that you already have open. The browser already has the manifest, the origin is already in your history, and you decide to install the app after you decide to visit the site.

Our Add dialog does the opposite. A person types or pastes an address, and the interface must then get the manifest of that origin to know if it can offer the app option. Thus a settings page makes a network request to an arbitrary address, and possibly to an address from an attacker, before the user agrees to anything.

These consequences are important:

- The request occurs on a page where users do not expect network activity, and a paste operation is the usual method to put an address into that field.
- The request tells the server that this profile will possibly add the site. If the request occurs at each `input` event and not at the submission, it gives that information at the speed of the typing.
- The request runs in the container that the dialog will use, or in no container, and this needs a decision.

These are the possible corrections, in the order of preference. First, make the request only at the submission and not during the typing. Second, offer the app option only for origins that are already in the history. Third, remove the app path from the dialog, and let the installation occur from the button in the address bar, where the browser already has the page. The dialog then makes only shortcuts. The third correction is the most defensible, and it has the lowest cost.

The prototype simulates this function with a static table. Thus the question is invisible in the mockup, and it would occur immediately in an implementation.

---

## 7. Two defects in the prototype, found by test

A test on the true DOM confirms both defects.

### 7.1 The duplicate check refuses the same app in two containers

```
const duplicate = state.apps.find(a => a.origin === url.origin && a.path === path);
```

The check ignores the container. Thus Firefox refuses Outlook in Work and Outlook in Personal with the message "Microsoft Outlook already opens this address."

This defect breaks the one capability that this design has and Chromium does not have, in the exact scenario that containers exist for. The key must be the origin, the path, and the container. The defect also relates to §5: after the correction, two apps with the same name exist, and the design must then answer the question about the name.

### 7.2 A tab that is not pinned can get a new address with no confirmation

The `switch-to-app` action goes through the save confirmation only when the app is pinned. The reason was that a new build of the shortcut can cause the loss of the pin. But the same action also changes the address that the app opens. For a tab that is not pinned, that change occurs with one click and only a toast. A test confirms this: a shortcut at `/gallery` became `/` with no message.

The confirmation uses the incorrect condition. The loss of a pin is the smaller consequence. A change to the address that the app opens is the larger consequence, and it must cause a confirmation on its own.

---

## 8. Manifest members that the design does not model

The design does not have to build each of these members. But it must have a position on each of them, because several members make state in the operating system that a removal must clean.

| Member | Chromium | Security relevance | Position that is necessary |
|---|---|---|---|
| `id` | The identity of the app after a change to `start_url` | High. It shows the difference between a new name and a substitution. | §4.12 uses it, but §9 does not define the meaning of a changed `id` |
| `display` / `display_override` | standalone, minimal-ui, tabbed, window-controls-overlay | High. It sets the quantity of chrome, thus the quantity of origin signal. | Not defined, see §1 |
| `file_handlers` | Writes ProgIDs into the Windows registry | High. A claim on file types at the level of the operating system. | The removal must remove the registration. "Deletes the shortcut" in §4.10 becomes untrue. |
| `protocol_handlers` | Registers URL schemes | High. A site claims `mailto:` or a custom scheme. | The same |
| `share_target` | Receives shared content | Medium | Not modelled |
| `shortcuts` | Jump list items | Low | Not modelled |
| `launch_handler` | focus-existing, navigate-existing, auto | Medium. It interacts with the link capture and with containers. | Not modelled |
| Badging API | Unread counts on the taskbar icon | Low | Not modelled |
| `scope_extensions` | Apps with more than one origin | High, see §4 | It stops a correct use case today |

The handler members are the important members. The Chromium documentation says clearly that an uninstall operation removes the registration of the file extension handlers. If Firefox registers handlers, the removal dialog must give each handler. The current text promises that only a shortcut goes away, and that promise would then be an untrue statement about the removal.

---

## 9. An enterprise device is a third source

Chromium has `WebAppInstallForceList`, which installs apps silently and prevents an uninstall operation by the user. It also has `WebAppSettings` for the policy of each app, and this includes `run_on_os_login`.

Our pair of badges is **From the site** and **Added by you**. A forced app is neither of these. It also breaks assumptions in other locations. **Remove Taskbar Tab** must be unavailable, and not only discouraged. The interface must also give the reason. If it does not, the disabled control is exactly the dead end that §4.2 refuses.

This document already records this item as an open question. It is a larger gap than that description shows, because it changes the badge vocabulary from a pair to a set. It also adds the first case where an action is truly unavailable.

---

## 10. Where the design is in front, and must stay in front

These items need a statement, because several of them are the reason to close the gaps above and not to stop the work.

| Capability | Chromium | This design |
|---|---|---|
| Change the name of an installed app locally | No. This is a long Edge request. | Yes, with the name from the site always visible and a reset control |
| Edit the address in the area of the app | No | Yes, with validation |
| Refuse a new name from a developer | Accept it, or leave it pending | Take it, and **Keep calling it {old}** writes a local name |
| Containers | No equivalent in a profile | Yes |
| Search, sort, multi-select, bulk remove | No | Yes |
| Honest permissions for each app | Shows them for each app, which implies the extent of the app | Read-only, stated as site-wide, with a link |
| One list for each source | Divided between `chrome://apps` and shortcuts | One list, with badges |

The decision about the permissions needs a specific defence. Chromium shows the permissions on a settings page for each app. Thus the permission appears to belong to the app. It does not. It belongs to the origin, and it applies in usual tabs also. A read-only display with a statement is the more truthful design. This design must not trade it for parity.

---

## 11. What to correct, in order

Each item below now has a decision. This document records the result, and the specification holds the behaviour.

| # | Item | Result | Location |
|---|---|---|---|
| 1 | Define the window | The origin is always present in true chrome. There is a different state outside the area, and an insecure state in the same position. This goes past Chromium, which shows nothing in the area, because an indicator that is usually absent is the easiest type to copy. | Spec §10 |
| 2 | Secure context | Firefox takes a manifest identity only over HTTPS. `http://` shortcuts stay permitted. The window carries the warning, and the list says nothing. | Spec §9, §10 |
| 3 | Duplicate check | The key is the address **and** the container. A second copy is permitted and must have a different name. The dialog gives a suggestion that follows the container. | Spec §5.1. **Corrected** |
| 4 | Confirmation for a new address | The interface always confirms. The loss of a pin was the smaller consequence and the incorrect condition. | Spec §4.11. **Corrected** |
| 5 | Link capture | It uses the declared area for manifest apps, and the host only when nothing is declared. The text gives the coarseness of the host rule. | Spec §4.7 |
| 6 | Identity key | The manifest `id` and the container. One copy holds the capture, and you select it explicitly. The start at sign-in stays a property of each copy. | Spec §9 |
| 7 | The area as a set | Firefox stores a list of matchers with a length of one today. Thus `scope_extensions` extends the check and does not replace it. | Spec §9 |
| 8 | The request in the add flow | Removed. The dialog makes shortcuts. The installation occurs from the address bar, where the browser already has the page. | Spec §5.1, §1.1 |
| 9 | Handler members | Not built, and this document records the liability in the removal text against the day that they come. | Spec §9, §4.10 |
| 10 | Enterprise devices | A third source, **Installed by {authority}**. The removal control is absent and not disabled, and the row gives the name of the authority. | Spec §3.1, §4.10 |

Item 1 is the one location where this design intentionally goes away from Chromium and does not follow it. Each other item is parity, or a consequence of containers, which Chromium has no equivalent of.

---

## Sources

- [Improvements to web app updates](https://developer.chrome.com/blog/improvements-to-web-app-updates), Chrome for Developers
- [Web App Scope Extensions](https://developer.chrome.com/docs/capabilities/scope-extensions), Chrome for Developers
- [Navigation management into installed PWAs](https://developer.chrome.com/docs/capabilities/pwa-navigation-management), Chrome for Developers
- [Windows Progressive Web App integration](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/windows_pwa_integration.md), Chromium docs
- [Security risks in web app off-scope navigation](https://github.com/w3c/manifest/issues/747), w3c/manifest
- [Web app manifest `scope`](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Manifest/Reference/scope), MDN
- [Making PWAs installable](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/Making_PWAs_installable), MDN
- [WebAppInstallForceList](https://chromeenterprise.google/policies/web-app-install-force-list/), Chrome Enterprise
