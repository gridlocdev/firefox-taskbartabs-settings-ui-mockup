# Firefox Taskbar Tabs: Settings UI Mockup

Firefox can change a website into a **Taskbar Tab**. A Taskbar Tab has its own window and its own icon on the taskbar. But you cannot manage the Taskbar Tabs that you add. There is no list, no control to change a name, and no control to change the page that a Taskbar Tab opens.

This project is a design prototype of that settings page. The page is an item in the sidebar of the new Firefox Settings. It is directly below Tabs and browsing.

## The files

| File | Description |
| --- | --- |
| [mockup.html](mockup.html) | The prototype. Open it in a browser. There is no build step and there are no dependencies. |
| [taskbar-tabs-options-specification.md](taskbar-tabs-options-specification.md) | Each option, the function of each of its states, and its behaviour in unusual conditions. |
| [taskbar-tabs-design-rationale.md](taskbar-tabs-design-rationale.md) | The reason for each decision. |
| [taskbar-tabs-provenance-analysis.md](taskbar-tabs-provenance-analysis.md) | One decision in full: how the interface shows the difference between an app that the site declares and a shortcut that you make. The analysis uses Nielsen's ten usability heuristics. |
| [taskbar-tabs-security-analysis.md](taskbar-tabs-security-analysis.md) | The questions that this design must answer, compared with the equivalent Chromium functions for installed web apps. |

## The main page

Three global preferences are at the top of the page. Below them, the page shows each Taskbar Tab that you added. Each row shows the address of the tab and the time of its last use. Each row also shows badges: if the tab is pinned, which container it uses, and if it opens at login. The search, sort, and multi-select controls help you when the list becomes long.

![The Taskbar Tabs settings page, with six installed apps](docs/screenshots/list-view.png)

This is the same page in the dark theme. The dark theme has no added cost, because each colour comes from an Acorn token.

![The same Taskbar Tabs settings page in the dark theme](docs/screenshots/list-view-dark.png)

The actions for each row are in an overflow menu. You can open the tab, change its name, copy its address, or go to its full settings.

![Row overflow menu with Open, Rename, Copy address, App settings, Clear data and Remove](docs/screenshots/row-menu.png)

To add a Taskbar Tab, you give only an address, a name, and a container.

![The Add a Taskbar Tab dialog](docs/screenshots/add-dialog.png)

## Apps that the site declares, and shortcuts that you make

Two types of item can be in this list. Most items are shortcuts: you selected the address and the name. The other items are apps that the site declares in a manifest. For these apps, the name, the icon, the start page, and the area belong to the developer. The name can also change when the site changes.

A third type occurs on a managed device: apps that a policy installs. These apps also take their identity from a manifest, but you cannot remove them.

The three types share one list. A person who looks for Teams must not have to know how Teams came into the list. A badge shows the difference, and the badge controls what you can edit. You can always change the name. The site's name stays below the field, and a reset control is adjacent to it. But the address must stay in the area that the app declares.

If an address is outside that area, the interface refuses it. It gives the rule and an alternative. This is better than a field that does not accept text and does not give a reason.

Containers can run one site two times. Thus an app can exist one time in each container. The second copy must have a different name. This prevents two identical icons on the taskbar, when one of the two icons holds your work account.

![The address field refuses an address that is outside the area of the app](docs/screenshots/spec/4.2-out-of-scope.png)

![The Add dialog asks for a different name for a second copy](docs/screenshots/spec/5.1-second-copy.png)

The full reasoning and the analysis against Nielsen's ten heuristics are in [taskbar-tabs-provenance-analysis.md](taskbar-tabs-provenance-analysis.md). The remaining questions after that analysis, compared with Chromium, are in [taskbar-tabs-security-analysis.md](taskbar-tabs-security-analysis.md).

## The page for each app

Each app has its own sub-page. On this page you can change the name and change the address that the app opens. You can also select if the app starts at sign-in and if it captures links, set its container, examine the site permissions, delete its data, or remove it.

![The detail sub-page for Home Assistant](docs/screenshots/detail-view.png)

## Confirmations that tell you what will occur

Each action that has a secondary effect tells you about that effect before you agree to it.

When you change the name or the address, Firefox makes the shortcut again. Thus the dialog shows the exact URL that the app will open. It also warns you that it is possible that you must pin the app again.

![Save changes dialog with the new URL and a warning about the pin](docs/screenshots/save-changes-dialog.png)

When you delete the site data, the dialog tells you how much data is in storage. It also tells you that the operation affects other tabs in the same container, and that the site will sign you out.

![Clear cookies and site data confirmation dialog](docs/screenshots/clear-data-dialog.png)

The control that removes an app is separate from the control that deletes its data. Thus the site stays available as a usual tab, if you do not select the option to delete its cookies.

![Remove Taskbar Tab confirmation dialog](docs/screenshots/remove-dialog.png)

## How to use the prototype

```
open mockup.html
```

Each control operates, because the prototype keeps its own state. You can add apps, edit them, cancel removals, and change between the light and the dark themes. The layout also adjusts to narrow windows.

## Status

This is a prototype, not production code. The target is the Windows desktop. Linux behaviour follows where Firefox gives support. This design does not include macOS.

## License

Mozilla Public License 2.0. The full text is in [LICENSE](LICENSE).
