# Security and feature gaps, measured against Chromium

A review of the design as it stands, against what Chromium ships for installed web apps. It covers what is missing, what is wrong, and what is deliberately different and should stay that way.

**Headline:** the identity work in [the provenance analysis](taskbar-tabs-provenance-analysis.md) is sound, but it guards the wrong end of the problem on its own. The address field in Settings is the surface an attacker needs least. The unguarded surface is the window at runtime, and the design says nothing about it at all.

**Status:** all ten findings have been resolved. Two were defects and are fixed in [mockup.html](mockup.html); the rest are decisions now carried by the [specification](taskbar-tabs-options-specification.md). §11 records where each one landed. The sections below are kept as the reasoning that produced those decisions, and are written in the present tense of the review.

---

## 1. The window is undefined, and it is where the security lives

Every document here describes a list, a sub-page and some dialogs. None of them describes the thing those settings produce: a browser window with reduced chrome, holding a site's cookies and permissions, wearing that site's icon, launched from the taskbar.

That window has no address bar. Removing the address bar removes the primary anti-phishing affordance in the browser, and the design has no position on what replaces it.

Chromium's position is explicit. When a standalone app window navigates off-origin it shows a mini URL bar carrying the new origin, keeping standalone display mode but making the boundary visible. The manifest spec asks for exactly this: a prominent UI element showing at least the origin, visually distinct from the in-scope state, so that leaving scope is obvious.

**What needs deciding:**

| Question | Chromium's answer | Ours |
|---|---|---|
| What identifies the origin while in scope? | Origin chip in the title bar, "App info" | Undefined |
| What happens on off-scope navigation? | Mini URL bar appears with the new origin | Undefined |
| Can the page counterfeit that indicator? | Partly, and it is a known open issue | Undefined |
| Does an insecure origin look different? | Yes, standard insecure indicator | Undefined |

The counterfeiting problem is worth naming rather than inheriting silently. [w3c/manifest#747](https://github.com/w3c/manifest/issues/747) records it: a page rendered in standalone mode can draw a convincing fake of the browser's own off-scope indicator, so the indicator is a hint and not a guarantee. Firefox's advantage here is that it can put the indicator in real window chrome that content cannot paint over, which is a genuine design opportunity rather than a gap to copy.

This is the single largest omission. The scope constraint on the address field (§4.2) stops a user or a support scam from repointing an app in Settings. It does nothing about the app navigating itself somewhere else the moment it opens, which is the cheaper attack and the one that does not require touching Settings at all.

---

## 2. Manifest apps need a secure context, and nothing enforces it

Chromium will not install a web app from a non-secure origin. HTTPS, or `localhost`, or nothing. This is not incidental: a manifest served over plain HTTP is attacker-modifiable in transit, and an app's identity is exactly what you do not want a network attacker editing.

The prototype accepts `http://` for both kinds of entry. Confirmed by test: `http://insecure.example/login` is accepted by the Add dialog without comment.

The rule should be split the way Chromium splits it:

- **From the site**: requires a secure context, because the identity claim is only worth something if the transport is authenticated.
- **Added by you**: `http://` allowed, since a shortcut to a local device is a real use case and the seed data already leans on it with `http://homeassistant.local:8123`. But the window must then carry the insecure indicator, which returns to §1.

Note the tension in the current seed data: Home Assistant is a plain-HTTP app that many people genuinely run, and it would be perverse to refuse it a shortcut. The distinction resolves it cleanly, which is a point in favour of having two kinds of entry at all.

---

## 3. Link capture is by host, where Chromium's is by scope

§4.7 matches by host, and the spec says so plainly: "Matching is by host, so it covers the whole site, not just the configured address."

Chromium captures navigations that fall within the manifest's declared scope.

Host matching is over-broad in a way that matters on shared hosts. An app installed from `someuser.github.io` would capture every link to `github.io`. An app on `tenant.atlassian.net` captures the host, which is fine, but `sharepoint.com` or any multi-tenant host is not. The failure mode is a link to someone else's content opening inside an app window that is branded as, and holds the cookies of, a different tenant.

For manifest apps the fix is free, because the scope is already declared and already parsed for the address field. For manual shortcuts there is no declared scope, so host matching is the only option available, and the honest framing is that manual shortcuts capture more coarsely than apps do.

This is already recorded in the rationale as an open question about separating start page from capture scope. It should be re-recorded as a security question, not an ergonomics one.

---

## 4. `scope_extensions` would break the address field

Chromium ships `scope_extensions`, which lets an app claim additional origins, each of which must opt in by hosting `/.well-known/web-app-origin-association` naming the app. Neither side can claim the relationship unilaterally. It exists because real apps span origins, and without it those navigations trip the off-scope bar and lose link capture.

Our `inScope` check is a single-origin comparison. An app that legitimately spans `example.com` and `example.co.uk` would have its own second origin rejected by the address field, with an error telling the user to add a separate Taskbar Tab.

The design does not have to implement scope extensions. It does have to not hard-code the assumption that scope is one origin, because the check is the thing that would need rewriting rather than extending. Treat the app's area as a set of matchers, currently of size one.

---

## 5. Containers and manifest identity collide, and this is unresolved

This is the part with no precedent to copy, because Chromium's isolation unit is a profile and Firefox's is a container inside a profile.

Chromium keys an installed app on its manifest `id` within a profile, so one app is one entry with one set of OS registrations. Two containers running the same app inside one Firefox profile means two entries sharing one manifest `id`, and several things then have no defined answer:

| Collision | Consequence |
|---|---|
| Link capture | A captured link matches two windows. Which container gets it? |
| Run at sign-in | Both start, or one does. Undefined |
| File and protocol handlers | OS registration is one ProgID per app. Two containers cannot both own `.pdf` |
| Shortcut identity | Two shortcuts, same manifest name and icon, different cookie jars, nothing on the taskbar to tell them apart |

The last one is a usability problem that becomes a security problem: two identical taskbar icons where one is your work account and one is not, and no way to tell which you clicked.

At minimum the design should state that the identity key is (manifest `id`, container) rather than manifest `id`, and that a container-differentiated app needs something visible to distinguish it. The container badge exists in the list already. It does not exist on the taskbar, in Alt+Tab, or in the Start menu, which is where the confusion actually happens. The local rename affordance is the available answer, and the design could reasonably prompt for a name when adding a second copy of the same app in a different container.

---

## 6. The Add dialog introduces a fetch that Chromium never performs

Chromium installs from a page you already have open. The manifest is already fetched, the origin is already in your history, and the decision to install comes after the decision to visit.

Our Add dialog inverts that. Someone types or pastes an address, and the interface has to fetch that origin's manifest to know whether to offer the app option. That is a settings page making a network request to an arbitrary, possibly attacker-supplied address, before the user has committed to anything.

Consequences worth weighing:

- The request happens on a page where users do not expect network activity, and paste is the normal way to get an address into that field.
- It confirms to the receiving server that this profile is about to add the site, and it leaks that at typing speed if the lookup is on `input` rather than on submit.
- It runs in whatever container the dialog is about to use, or none, which needs deciding.

Mitigations, roughly in order of preference: fetch only on submit rather than while typing; only offer the app option for origins already in history; or drop the app path from the dialog entirely and let installation happen from the address bar button, where the page is already loaded, leaving the dialog to make shortcuts only. The last is the most defensible and costs the least.

The prototype fakes this with a static lookup table, so the question is invisible in the mockup and would surface immediately in implementation.

---

## 7. Two defects in the prototype, found by testing

Both confirmed by driving the real DOM.

### 7.1 Duplicate detection blocks the same app in two containers

```
const duplicate = state.apps.find(a => a.origin === url.origin && a.path === path);
```

The check ignores the container. Adding Outlook in Work and Outlook in Personal is refused with "Microsoft Outlook already opens this address."

This breaks the one capability this design has that Chromium does not, in exactly the scenario that motivates containers. The key should be origin, path and container. It also intersects §5: once permitted, two identically named apps exist, which is when the naming question has to be answered.

### 7.2 An unpinned app can be re-addressed with no confirmation

`switch-to-app` routes through the save confirmation only when the app is pinned, on the reasoning that rebuilding the shortcut may cost the pin. But the same action also changes the address the app opens, and for an unpinned app that happens on one click with only a toast to show for it. Confirmed: a shortcut at `/gallery` became `/` silently.

The confirmation is currently gated on the wrong thing. Losing a pin is the lesser consequence; changing what the app opens is the greater one, and it should confirm on its own account.

---

## 8. Manifest members the design does not model

Not all of these need building. They need a stated position, because several create OS-level state that removal has to clean up.

| Member | Chromium | Security relevance | Position needed |
|---|---|---|---|
| `id` | App identity across `start_url` changes | High. Distinguishes a rename from a substitution | §4.12 relies on it but §9 does not define what a changed `id` means |
| `display` / `display_override` | standalone, minimal-ui, tabbed, window-controls-overlay | High. Determines how much chrome, so how much origin signal | Undefined, see §1 |
| `file_handlers` | Registers ProgIDs in the Windows registry | High. OS-level claim on file types | Removal must unregister. §4.10's "deletes the shortcut" becomes untrue |
| `protocol_handlers` | Registers URL schemes | High. A site claiming `mailto:` or a custom scheme | Same |
| `share_target` | Receives shared content | Medium | Not modelled |
| `shortcuts` | Jump list entries | Low | Not modelled |
| `launch_handler` | focus-existing, navigate-existing, auto | Medium. Interacts with link capture and containers | Not modelled |
| Badging API | Unread counts on the taskbar icon | Low | Not modelled |
| `scope_extensions` | Multi-origin apps | High, see §4 | Blocks a legitimate case today |

The handler members are the important ones. Chromium's documentation is explicit that uninstall unregisters file extension handlers. If Firefox ever registers handlers, the removal dialog has to enumerate them, and the current copy promising that only a shortcut goes away would be a false statement about what removal did.

---

## 9. Enterprise is a third provenance

Chromium has `WebAppInstallForceList`, which installs apps silently and prevents the user uninstalling them, and `WebAppSettings` for per-app policy including `run_on_os_login`.

Our badge pair is **From the site** and **Added by you**. A force-installed app is neither. It also breaks assumptions elsewhere: **Remove Taskbar Tab** must be unavailable rather than merely discouraged, and the reason has to be stated or the disabled control is exactly the dead end §4.2 argues against.

This is recorded as an open question already. It is a bigger hole than that framing suggests, because it changes the badge vocabulary from a pair to a set and adds the first case where an action is genuinely unavailable.

---

## 10. Where the design is ahead, and should stay ahead

Worth stating, because several of these are the reason the gaps above are worth closing rather than abandoning.

| Capability | Chromium | Here |
|---|---|---|
| Rename an installed app locally | No. Long-standing Edge request | Yes, with the site's name always visible and a reset |
| Edit the address within scope | No | Yes, validated |
| Decline a developer's rename | Accept or leave pending | Adopt, with **Keep calling it {old}** writing a local name |
| Containers | No equivalent inside a profile | Yes |
| Search, sort, multi-select, bulk remove | No | Yes |
| Honest per-app permissions | Shows them per-app, implying app scope | Shown read-only, stated as site-scoped, linked out |
| One list regardless of provenance | Split across `chrome://apps` and shortcuts | One list, badged |

The permissions decision deserves particular defence. Chromium surfaces permissions on a per-app settings page, which reads as though the permission belongs to the app. It does not; it belongs to the origin, and it applies in ordinary tabs too. Being read-only and saying so is the more truthful design and should not be traded away for parity.

---

## 11. What to address, in order

Every item below has since been decided. The resolution is recorded here; the behaviour lives in the specification.

| # | Item | Resolution | Where |
|---|---|---|---|
| 1 | Define the window | Origin always present in real chrome, distinct state off-scope, insecure state in the same slot. Goes past Chromium, which shows nothing in scope, because an indicator that is usually absent is the easiest to counterfeit | Spec §10 |
| 2 | Secure context | Manifest identity is only taken over HTTPS. `http://` shortcuts stay allowed; the window carries the warning and the list says nothing | Spec §9, §10 |
| 3 | Duplicate detection | Keyed on address **and** container. A second copy is allowed and required to have a distinct name, prefilled and following the container | Spec §5.1. **Fixed** |
| 4 | Re-address confirmation | Always confirms. Losing a pin was the lesser consequence and the wrong thing to gate on | Spec §4.11. **Fixed** |
| 5 | Link capture | Scope-based for manifest apps, host-based only where nothing was declared, and the coarseness is stated in the copy | Spec §4.7 |
| 6 | Identity key | Manifest `id` plus container. One capture holder chosen explicitly; sign-in startup stays per-copy | Spec §9 |
| 7 | Area as a set | Stored as a list of matchers, length one today, so `scope_extensions` extends rather than rewrites | Spec §9 |
| 8 | Add-flow fetch | Removed. The dialog makes shortcuts; installing happens from the address bar, where the page is already loaded | Spec §5.1, §1.1 |
| 9 | Handler members | Explicitly not built, with the removal-copy liability recorded against the day they land | Spec §9, §4.10 |
| 10 | Enterprise | Third provenance, **Installed by {authority}**. Removal is absent rather than disabled, and the row names who decides | Spec §3.1, §4.10 |

The one place this design deliberately diverges from Chromium rather than catching up is item 1. Everything else is either parity or a consequence of containers, which Chromium has no equivalent of.

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
